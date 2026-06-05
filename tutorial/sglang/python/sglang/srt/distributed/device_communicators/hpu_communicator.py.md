# hpu_communicator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/hpu_communicator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `hpu_communicator`. It exposes primary entry points such as `HpuCommunicator`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `hpu_communicator` 的逻辑。 它对外提供的主要入口包括 `HpuCommunicator`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/device_communicators/hpu_communicator.py

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from sglang.srt.utils import is_hpu

if is_hpu():
    import habana_frameworks.torch as htorch  # noqa: F401


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 15-16: Class HpuCommunicator
```python
class HpuCommunicator:

```
**EN:** This range introduces `HpuCommunicator` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `HpuCommunicator`，并定义其后续方法依赖的结构或元数据。

### Lines 17-23: Method HpuCommunicator.__init__
```python
    def __init__(self, group: ProcessGroup):
        if not is_hpu():
            self.disabled = True
            return
        self.disabled = False
        self.group = group
        self.world_size = dist.get_world_size(self.group)
```
**EN:** This callable implements `HpuCommunicator.__init__`. It takes `group` and mainly initializes instance state and defaults. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `HpuCommunicator.__init__`。它接收 `group`，主要用于初始化实例状态与默认值。 在这一范围内，它会协调分布式通信。

### Lines 25-31: Method HpuCommunicator.all_reduce
```python
    def all_reduce(self, x: torch.Tensor) -> torch.Tensor:
        # FIXME(kzawora): this is a workaround for a bug in Habana PT bridge
        # occurring when PT_HPU_ENABLE_LAZY_COLLECTIVES=true env var is used
        # (which is required for tensor parallel HPUGraph inference)
        htorch.core.mark_step()
        dist.all_reduce(x, group=self.group)
        return x
```
**EN:** This callable implements `HpuCommunicator.all_reduce`. It takes `x` and mainly reduces or aggregates values. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `HpuCommunicator.all_reduce`。它接收 `x`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 33-51: Method HpuCommunicator.all_gather
```python
    def all_gather(self, x: torch.Tensor, dim: int = -1) -> torch.Tensor:
        world_size = self.world_size
        if dim < 0:
            # Convert negative dim to positive.
            dim += x.dim()
        input_size = x.size()
        # Allocate output tensor.
        output_tensor = torch.empty(
            (world_size,) + input_size, dtype=x.dtype, device=x.device
        )
        # All-gather.
        htorch.core.mark_step()
        dist.all_gather_into_tensor(output_tensor, x, group=self.group)
        # Reshape
        output_tensor = output_tensor.movedim(0, dim)
        output_tensor = output_tensor.reshape(
            input_size[:dim] + (world_size * input_size[dim],) + input_size[dim + 1 :]
        )
        return output_tensor
```
**EN:** This callable implements `HpuCommunicator.all_gather`. It takes `x`, `dim` and mainly implements all gather.
**CN:** 这一可调用对象实现了 `HpuCommunicator.all_gather`。它接收 `x`, `dim`，主要用于实现 all gather 相关逻辑。

## Key Concepts / 关键概念
- `HpuCommunicator`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Third-party / 第三方**: `torch`, `torch.distributed`, `habana_frameworks.torch`
- **Internal modules / 内部模块**: `sglang.srt.utils`
