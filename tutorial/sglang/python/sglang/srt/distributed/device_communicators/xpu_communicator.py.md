# xpu_communicator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/xpu_communicator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `xpu_communicator`. It exposes primary entry points such as `XpuCommunicator`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `xpu_communicator` 的逻辑。 它对外提供的主要入口包括 `XpuCommunicator`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/device_communicators/xpu_communicator.py

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from sglang.srt.utils import is_xpu


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 12-13: Class XpuCommunicator
```python
class XpuCommunicator:

```
**EN:** This range introduces `XpuCommunicator` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `XpuCommunicator`，并定义其后续方法依赖的结构或元数据。

### Lines 14-20: Method XpuCommunicator.__init__
```python
    def __init__(self, group: ProcessGroup):
        if not is_xpu():
            self.disabled = True
            return
        self.disabled = False
        self.group = group
        self.world_size = dist.get_world_size(self.group)
```
**EN:** This callable implements `XpuCommunicator.__init__`. It takes `group` and mainly initializes instance state and defaults. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `XpuCommunicator.__init__`。它接收 `group`，主要用于初始化实例状态与默认值。 在这一范围内，它会协调分布式通信。

### Lines 22-24: Method XpuCommunicator.all_reduce
```python
    def all_reduce(self, x: torch.Tensor) -> torch.Tensor:
        dist.all_reduce(x, group=self.group)
        return x
```
**EN:** This callable implements `XpuCommunicator.all_reduce`. It takes `x` and mainly reduces or aggregates values. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `XpuCommunicator.all_reduce`。它接收 `x`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 26-50: Method XpuCommunicator.gather
```python
    def gather(
        self, input_: torch.Tensor, rank_in_group: int, dst: int = 0, dim: int = -1
    ):
        # For xpu path, gather doesn't work properly together with ray
        # cluster so we use all_gather instead for now.
        input_size = input_.size()
        # Allocate output tensor.
        output_tensor = torch.empty(
            (self.world_size,) + input_size, dtype=input_.dtype, device=input_.device
        )
        # All-gather.
        torch.distributed.all_gather_into_tensor(
            output_tensor, input_, group=self.group
        )
        if rank_in_group == dst:
            # Reshape
            output_tensor = output_tensor.movedim(0, dim)
            output_tensor = output_tensor.reshape(
                input_size[:dim]
                + (self.world_size * input_size[dim],)
                + input_size[dim + 1 :]
            )
        else:
            output_tensor = None
        return output_tensor
```
**EN:** This callable implements `XpuCommunicator.gather`. It takes `input_`, `rank_in_group`, `dst`, `dim` and mainly implements gather. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `XpuCommunicator.gather`。它接收 `input_`, `rank_in_group`, `dst`, `dim`，主要用于实现 gather 相关逻辑。 在这一范围内，它会协调分布式通信。

## Key Concepts / 关键概念
- `XpuCommunicator`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.srt.utils`
