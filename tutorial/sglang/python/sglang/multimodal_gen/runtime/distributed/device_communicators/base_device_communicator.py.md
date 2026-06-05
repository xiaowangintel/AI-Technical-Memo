# base_device_communicator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/device_communicators/base_device_communicator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `DistributedAutograd`, and `DeviceCommunicatorBase`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `DistributedAutograd` 和 `DeviceCommunicatorBase` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 7-12: module setup and imports / 模块初始化与导入
```python
from typing import Any

import torch
import torch.distributed as dist
from torch import Tensor
from torch.distributed import ProcessGroup, ReduceOp
```
**EN:** This block establishes the module context and imports `typing`, `torch`, and `torch.distributed`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `typing`、`torch` 和 `torch.distributed`。这些依赖为后续实现提供所需符号。

### Lines 15-22: `DistributedAutograd` class overview / `DistributedAutograd` 类概览
```python
class DistributedAutograd:
    """Collection of autograd functions for distributed operations.

    This class provides custom autograd functions for distributed operations like all_reduce,
    all_gather, and all_to_all. Each operation is implemented as a static inner class with
    proper forward and backward implementations.
    """
```
**EN:** This block defines class `DistributedAutograd`. Collection of autograd functions for distributed operations. This class provides custom autograd functions for distributed operations like all_reduce, all_gather, and all_to_all.
**CN:** 该代码块定义了类 `DistributedAutograd`。 它用于封装 distributed autograd 相关行为。

### Lines 23-47: `AllReduce` class definition / `AllReduce` 类定义
```python
    class AllReduce(torch.autograd.Function):
        """Differentiable all_reduce operation.

        The gradient of all_reduce is another all_reduce operation since the operation
        combines values from all ranks equally.
        """

        @staticmethod
        def forward(
            ctx: Any,
            group: ProcessGroup,
            input_: Tensor,
            op: dist.ReduceOp | None = None,
        ) -> Tensor:
            ctx.group = group
            ctx.op = op
            output = input_.clone()
            dist.all_reduce(output, group=group, op=op)
            return output

        @staticmethod
        def backward(ctx: Any, grad_output: Tensor) -> tuple[None, Tensor, None]:
            grad_output = grad_output.clone()
            dist.all_reduce(grad_output, group=ctx.group, op=ctx.op)
            return None, grad_output, None
```
**EN:** This block defines method `AllReduce` on `DistributedAutograd`. Differentiable all_reduce operation. The gradient of all_reduce is another all_reduce operation since the operation combines values from all ranks equally. It inherits from `torch.autograd.Function`.
**CN:** 该代码块定义了 `DistributedAutograd` 的方法 `AllReduce`。 它用于封装 all reduce 相关行为。 它继承自 `torch.autograd.Function`。

### Lines 49-101: `AllGather` class definition / `AllGather` 类定义
```python
    class AllGather(torch.autograd.Function):
        """Differentiable all_gather operation.

        The operation gathers tensors from all ranks and concatenates them along a specified dimension.
        The backward pass uses reduce_scatter to efficiently distribute gradients back to source ranks.
        """

        @staticmethod
        def forward(
            ctx: Any, group: ProcessGroup, input_: Tensor, world_size: int, dim: int
        ) -> Tensor:
            ctx.group = group
            ctx.world_size = world_size
            ctx.dim = dim
            ctx.input_shape = input_.shape

            input_size = input_.size()
            output_size = (input_size[0] * world_size,) + input_size[1:]
            output_tensor = torch.empty(
                output_size, dtype=input_.dtype, device=input_.device
            )

            dist.all_gather_into_tensor(output_tensor, input_, group=group)

            output_tensor = output_tensor.reshape((world_size,) + input_size)
            output_tensor = output_tensor.movedim(0, dim)
            output_tensor = output_tensor.reshape(
                input_size[:dim]
                + (world_size * input_size[dim],)
                + input_size[dim + 1 :]
            )
            return output_tensor

        @staticmethod
        def backward(ctx: Any, grad_output: Tensor) -> tuple[None, Tensor, None, None]:
            # Split the gradient tensor along the gathered dimension
            dim_size = grad_output.size(ctx.dim) // ctx.world_size
            grad_chunks = grad_output.reshape(
                grad_output.shape[: ctx.dim]
                + (ctx.world_size, dim_size)
                + grad_output.shape[ctx.dim + 1 :]
            )
            grad_chunks = grad_chunks.movedim(ctx.dim, 0)

            # Each rank only needs its corresponding gradient
            grad_input = torch.empty(
                ctx.input_shape, dtype=grad_output.dtype, device=grad_output.device
            )
            dist.reduce_scatter_tensor(
                grad_input, grad_chunks.contiguous(), group=ctx.group
            )

            return None, grad_input, None, None
```
**EN:** This block defines method `AllGather` on `DistributedAutograd`. Differentiable all_gather operation. The operation gathers tensors from all ranks and concatenates them along a specified dimension. It inherits from `torch.autograd.Function`.
**CN:** 该代码块定义了 `DistributedAutograd` 的方法 `AllGather`。 它用于封装 all gather 相关行为。 它继承自 `torch.autograd.Function`。

### Lines 103-196: `AllToAll4D` class definition / `AllToAll4D` 类定义
```python
    class AllToAll4D(torch.autograd.Function):
        """Differentiable all_to_all operation specialized for 4D tensors.

        This operation is particularly useful for attention operations where we need to
        redistribute data across ranks for efficient parallel processing.

        The operation supports two modes:
        1. scatter_dim=2, gather_dim=1: Used for redistributing attention heads
        2. scatter_dim=1, gather_dim=2: Used for redistributing sequence dimensions
        """

        @staticmethod
        def forward(
            ctx: Any,
            group: ProcessGroup,
            input_: Tensor,
            world_size: int,
            scatter_dim: int,
            gather_dim: int,
        ) -> Tensor:
            ctx.group = group
            ctx.world_size = world_size
            ctx.scatter_dim = scatter_dim
            ctx.gather_dim = gather_dim

            if world_size == 1:
                return input_

            assert (
                input_.dim() == 4
            ), f"input must be 4D tensor, got {input_.dim()} and shape {input_.shape}"

            if scatter_dim == 2 and gather_dim == 1:
                bs, shard_seqlen, hn, hd = input_.shape
                seqlen = shard_seqlen * world_size
                shard_hn = hn // world_size

                input_ = input_.transpose(0, 2).contiguous()  # hn, shard_seqlen, bs, hd
                output = torch.empty_like(input_)

                dist.all_to_all_single(
                    output, input_, group=group
                )  # hn, shard_seqlen, bs, hd

                output = torch.cat(
                    output.split(shard_hn), dim=1
                )  # sharded hn, seqlen, bs, hd

                output = output.transpose(
                    0, 2
                ).contiguous()  # bs, seqlen, sharded_hn, hd

                return output
            elif scatter_dim == 1 and gather_dim == 2:
                bs, seqlen, shard_hn, hd = input_.shape
                hn = shard_hn * world_size
                shard_seqlen = seqlen // world_size

                input_ = input_.transpose(0, 2).contiguous()  # shard_hn, seqlen, bs, hd

                input_ = (
                    input_.reshape(shard_hn, world_size, shard_seqlen, bs, hd)
                    .transpose(0, 1)
                    .reshape(shard_hn * world_size, shard_seqlen, bs, hd)
                    .contiguous()
                )

                output = torch.empty_like(input_)

                dist.all_to_all_single(output, input_, group=group)

                output = output.transpose(
                    0, 2
                ).contiguous()  # bs, seqlen, sharded_hn, hd

                return output
            else:
                raise RuntimeError(
                    f"Invalid scatter_dim={scatter_dim}, gather_dim={gather_dim}. "
                    f"Only (scatter_dim=2, gather_dim=1) and (scatter_dim=1, gather_dim=2) are supported."
                )

        @staticmethod
        def backward(
            ctx: Any, grad_output: Tensor
        ) -> tuple[None, Tensor, None, None, None]:
            if ctx.world_size == 1:
                return None, grad_output, None, None, None

            # For backward pass, we swap scatter_dim and gather_dim
            output = DistributedAutograd.AllToAll4D.apply(
                ctx.group, grad_output, ctx.world_size, ctx.gather_dim, ctx.scatter_dim
            )
            return None, output, None, None, None
```
**EN:** This block defines method `AllToAll4D` on `DistributedAutograd`. Differentiable all_to_all operation specialized for 4D tensors. This operation is particularly useful for attention operations where we need to redistribute data across ranks for efficient parallel processing. It inherits from `torch.autograd.Function`.
**CN:** 该代码块定义了 `DistributedAutograd` 的方法 `AllToAll4D`。 它用于封装 all to all4 d 相关行为。 它继承自 `torch.autograd.Function`。

### Lines 199-206: `DeviceCommunicatorBase` class overview / `DeviceCommunicatorBase` 类概览
```python
class DeviceCommunicatorBase:
    """
    Base class for device-specific communicator with autograd support.
    It can use the `cpu_group` to initialize the communicator.
    If the device has PyTorch integration (PyTorch can recognize its
    communication backend), the `device_group` will also be given.
    """
```
**EN:** This block defines class `DeviceCommunicatorBase`. Base class for device-specific communicator with autograd support. It can use the `cpu_group` to initialize the communicator.
**CN:** 该代码块定义了类 `DeviceCommunicatorBase`。 它用于封装 device communicator base 相关行为。

### Lines 207-223: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        cpu_group: ProcessGroup,
        device: torch.device | None = None,
        device_group: ProcessGroup | None = None,
        unique_name: str = "",
    ):
        self.device = device or torch.device("cpu")
        self.cpu_group = cpu_group
        self.device_group = device_group
        self.unique_name = unique_name
        self.rank = dist.get_rank(cpu_group)
        self.world_size = dist.get_world_size(cpu_group)
        self.ranks = dist.get_process_group_ranks(cpu_group)
        self.global_rank = dist.get_rank()
        self.global_world_size = dist.get_world_size()
        self.rank_in_group = dist.get_group_rank(self.cpu_group, self.global_rank)
```
**EN:** This block defines method `__init__` on `DeviceCommunicatorBase`. It initializes the instance state. Key calls include `dist.get_rank`, `dist.get_world_size`, `dist.get_process_group_ranks`, `dist.get_group_rank`, and `torch.device`. Parameters such as `cpu_group`, `device`, `device_group`, and `unique_name` drive the behavior in this section.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `dist.get_rank`、`dist.get_world_size`、`dist.get_process_group_ranks`、`dist.get_group_rank` 和 `torch.device`。 本段逻辑主要由 `cpu_group`、`device`、`device_group` 和 `unique_name` 等参数驱动。

### Lines 225-229: `all_reduce` implementation / `all_reduce` 实现
```python
    def all_reduce(
        self, input_: torch.Tensor, op: dist.ReduceOp | None = ReduceOp.SUM
    ) -> torch.Tensor:
        """Performs an all_reduce operation with gradient support."""
        return DistributedAutograd.AllReduce.apply(self.device_group, input_, op)
```
**EN:** This block defines method `all_reduce` on `DeviceCommunicatorBase`. Performs an all_reduce operation with gradient support. Key calls include `DistributedAutograd.AllReduce.apply`. Parameters such as `input_`, and `op` drive the behavior in this section.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `all_reduce`。 它用于处理 all reduce 相关逻辑。 关键调用包括 `DistributedAutograd.AllReduce.apply`。 本段逻辑主要由 `input_` 和 `op` 等参数驱动。

### Lines 231-237: `all_gather` implementation / `all_gather` 实现
```python
    def all_gather(self, input_: torch.Tensor, dim: int = -1) -> torch.Tensor:
        """Performs an all_gather operation with gradient support."""
        if dim < 0:
            dim += input_.dim()
        return DistributedAutograd.AllGather.apply(
            self.device_group, input_, self.world_size, dim
        )
```
**EN:** This block defines method `all_gather` on `DeviceCommunicatorBase`. Performs an all_gather operation with gradient support. Key calls include `DistributedAutograd.AllGather.apply`, and `input_.dim`. The implementation branches on conditions. Parameters such as `input_`, and `dim` drive the behavior in this section.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `all_gather`。 它用于处理 all gather 相关逻辑。 关键调用包括 `DistributedAutograd.AllGather.apply` 和 `input_.dim`。 实现中包含条件分支。 本段逻辑主要由 `input_` 和 `dim` 等参数驱动。

### Lines 239-245: `all_to_all_4D` implementation / `all_to_all_4D` 实现
```python
    def all_to_all_4D(
        self, input_: torch.Tensor, scatter_dim: int = 2, gather_dim: int = 1
    ) -> torch.Tensor:
        """Performs a 4D all-to-all operation with gradient support."""
        return DistributedAutograd.AllToAll4D.apply(
            self.device_group, input_, self.world_size, scatter_dim, gather_dim
        )
```
**EN:** This block defines method `all_to_all_4D` on `DeviceCommunicatorBase`. Performs a 4D all-to-all operation with gradient support. Key calls include `DistributedAutograd.AllToAll4D.apply`. Parameters such as `input_`, `scatter_dim`, and `gather_dim` drive the behavior in this section.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `all_to_all_4D`。 它用于处理 all to all 4 d 相关逻辑。 关键调用包括 `DistributedAutograd.AllToAll4D.apply`。 本段逻辑主要由 `input_`、`scatter_dim` 和 `gather_dim` 等参数驱动。

### Lines 247-276: `gather` implementation / `gather` 实现
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
        torch.distributed.gather(
            input_, gather_list, dst=self.ranks[dst], group=self.device_group
        )
        if self.rank_in_group == dst:
            output_tensor = torch.cat(gather_list, dim=dim)
        else:
            output_tensor = None
        return output_tensor
```
**EN:** This block defines method `gather` on `DeviceCommunicatorBase`. NOTE: We assume that the input tensor is on the same device across all the ranks. NOTE: `dst` is the local rank of the destination rank. Key calls include `torch.distributed.gather`, `input_.dim`, `torch.cat`, `input_.size`, and `torch.empty_like`. The implementation branches on conditions. Parameters such as `input_`, `dst`, and `dim` drive the behavior in this section.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `gather`。 它用于处理 gather 相关逻辑。 关键调用包括 `torch.distributed.gather`、`input_.dim`、`torch.cat`、`input_.size` 和 `torch.empty_like`。 实现中包含条件分支。 本段逻辑主要由 `input_`、`dst` 和 `dim` 等参数驱动。

### Lines 278-283: `send` implementation / `send` 实现
```python
    def send(self, tensor: torch.Tensor, dst: int | None = None) -> None:
        """Sends a tensor to the destination rank in a non-blocking way"""
        """NOTE: `dst` is the local rank of the destination rank."""
        if dst is None:
            dst = (self.rank_in_group + 1) % self.world_size
        torch.distributed.send(tensor, self.ranks[dst], self.device_group)
```
**EN:** This block defines method `send` on `DeviceCommunicatorBase`. Sends a tensor to the destination rank in a non-blocking way Key calls include `torch.distributed.send`. The implementation branches on conditions. Parameters such as `tensor`, and `dst` drive the behavior in this section.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `send`。 它用于处理 send 相关逻辑。 关键调用包括 `torch.distributed.send`。 实现中包含条件分支。 本段逻辑主要由 `tensor` 和 `dst` 等参数驱动。

### Lines 285-295: `recv` implementation / `recv` 实现
```python
    def recv(
        self, size: torch.Size, dtype: torch.dtype, src: int | None = None
    ) -> torch.Tensor:
        """Receives a tensor from the source rank."""
        """NOTE: `src` is the local rank of the source rank."""
        if src is None:
            src = (self.rank_in_group - 1) % self.world_size

        tensor = torch.empty(size, dtype=dtype, device=self.device)
        torch.distributed.recv(tensor, self.ranks[src], self.device_group)
        return tensor
```
**EN:** This block defines method `recv` on `DeviceCommunicatorBase`. Receives a tensor from the source rank. Key calls include `torch.empty`, and `torch.distributed.recv`. The implementation branches on conditions. Parameters such as `size`, `dtype`, and `src` drive the behavior in this section.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `recv`。 它用于处理 recv 相关逻辑。 关键调用包括 `torch.empty` 和 `torch.distributed.recv`。 实现中包含条件分支。 本段逻辑主要由 `size`、`dtype` 和 `src` 等参数驱动。

### Lines 297-298: `destroy` implementation / `destroy` 实现
```python
    def destroy(self) -> None:
        pass
```
**EN:** This block defines method `destroy` on `DeviceCommunicatorBase`. It handles destroy logic.
**CN:** 该代码块定义了 `DeviceCommunicatorBase` 的方法 `destroy`。 它用于处理 destroy 相关逻辑。

## Key Concepts / 关键概念
- `DistributedAutograd`: Collection of autograd functions for distributed operations. / 核心类，用于封装 distributed autograd 相关行为。
- `DeviceCommunicatorBase`: Base class for device-specific communicator with autograd support. / 核心类，用于封装 device communicator base 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`

- **Total lines / 总行数**: 298
