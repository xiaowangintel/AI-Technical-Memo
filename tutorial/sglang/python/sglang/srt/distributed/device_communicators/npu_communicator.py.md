# npu_communicator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/npu_communicator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `npu_communicator`. It exposes primary entry points such as `NpuCommunicator`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `npu_communicator` 的逻辑。 它对外提供的主要入口包括 `NpuCommunicator`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module imports, constants, and setup
```python
import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from sglang.srt.utils import is_npu

_is_npu = is_npu()

if _is_npu:
    from torch_npu import npu_dynamic_quant


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 13-14: Class NpuCommunicator
```python
class NpuCommunicator:

```
**EN:** This range introduces `NpuCommunicator` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `NpuCommunicator`，并定义其后续方法依赖的结构或元数据。

### Lines 15-21: Method NpuCommunicator.__init__
```python
    def __init__(self, group: ProcessGroup):
        if not _is_npu:
            self.disabled = True
            return
        self.disabled = False
        self.group = group
        self.world_size = dist.get_world_size(self.group)
```
**EN:** This callable implements `NpuCommunicator.__init__`. It takes `group` and mainly initializes instance state and defaults. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `NpuCommunicator.__init__`。它接收 `group`，主要用于初始化实例状态与默认值。 在这一范围内，它会协调分布式通信。

### Lines 23-25: Method NpuCommunicator.all_reduce
```python
    def all_reduce(self, x: torch.Tensor) -> torch.Tensor:
        dist.all_reduce(x, group=self.group)
        return x
```
**EN:** This callable implements `NpuCommunicator.all_reduce`. It takes `x` and mainly reduces or aggregates values. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `NpuCommunicator.all_reduce`。它接收 `x`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 27-52: Method NpuCommunicator.quant_all_reduce
```python
    def quant_all_reduce(self, x: torch.Tensor) -> torch.Tensor:
        """
        Note:
        All reduce is split into All gather + reduce.
        All gather is performed in low precision, but reduce in full precision.
        """
        world_size = self.world_size
        input_size = x.size()
        output_size = (input_size[0] * world_size,) + input_size[1:]
        x_q, scale = npu_dynamic_quant(x, dst_type=torch.int8)
        # Allocate output tensor.
        output_tensor = torch.empty(output_size, dtype=x_q.dtype, device=x.device)
        output_scale = torch.empty(
            output_size[:1], dtype=scale.dtype, device=scale.device
        )
        # All-gather.
        dist.all_gather_into_tensor(output_tensor, x_q, group=self.group)
        dist.all_gather_into_tensor(output_scale, scale, group=self.group)

        output_tensor = output_tensor.to(x.dtype) * output_scale.unsqueeze(-1).to(
            x.dtype
        )
        # Reshape
        output_tensor = output_tensor.reshape((world_size,) + input_size)

        return output_tensor.sum(dim=0)
```
**EN:** This callable implements `NpuCommunicator.quant_all_reduce`. It takes `x` and mainly reduces or aggregates values. The docstring states: "Note: All reduce is split into All gather + reduce." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `NpuCommunicator.quant_all_reduce`。它接收 `x`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 54-71: Method NpuCommunicator.all_gather
```python
    def all_gather(self, x: torch.Tensor, dim: int = -1) -> torch.Tensor:
        world_size = self.world_size
        if dim < 0:
            # Convert negative dim to positive.
            dim += x.dim()
        input_size = x.size()
        output_size = (input_size[0] * world_size,) + input_size[1:]
        # Allocate output tensor.
        output_tensor = torch.empty(output_size, dtype=x.dtype, device=x.device)
        # All-gather.
        dist.all_gather_into_tensor(output_tensor, x, group=self.group)
        # Reshape
        output_tensor = output_tensor.reshape((world_size,) + input_size)
        output_tensor = output_tensor.movedim(0, dim)
        output_tensor = output_tensor.reshape(
            input_size[:dim] + (world_size * input_size[dim],) + input_size[dim + 1 :]
        )
        return output_tensor
```
**EN:** This callable implements `NpuCommunicator.all_gather`. It takes `x`, `dim` and mainly implements all gather.
**CN:** 这一可调用对象实现了 `NpuCommunicator.all_gather`。它接收 `x`, `dim`，主要用于实现 all gather 相关逻辑。

## Key Concepts / 关键概念
- `NpuCommunicator`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Third-party / 第三方**: `torch`, `torch.distributed`, `torch_npu`
- **Internal modules / 内部模块**: `sglang.srt.utils`
