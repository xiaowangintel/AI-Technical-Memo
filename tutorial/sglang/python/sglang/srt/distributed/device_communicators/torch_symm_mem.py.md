# torch_symm_mem.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/torch_symm_mem.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `torch_symm_mem`. It exposes primary entry points such as `TorchSymmMemCommunicator`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `torch_symm_mem` 的逻辑。 它对外提供的主要入口包括 `TorchSymmMemCommunicator`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/bf214ca22625e311a2c4c0dfbf7af19128f4919c/vllm/distributed/device_communicators/symm_mem.py
import logging
from typing import Optional, Union

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from sglang.srt.distributed.device_communicators.all_reduce_utils import (
    TORCH_SYMM_MEM_ALL_REDUCE_MAX_SIZES,
)
from sglang.srt.utils import is_cuda, is_hip

try:
    import torch.distributed._symmetric_memory as torch_symm_mem

    _is_cuda = is_cuda()
    _is_hip = is_hip()

    torch_symm_mem_available = False
    if _is_cuda:
        torch_symm_mem_available = True
except ImportError:
    torch_symm_mem_available = False


logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 32-53: Class TorchSymmMemCommunicator
```python
class TorchSymmMemCommunicator:
    """
    Thin wrapper around torch-symmetric-memory collectives.

    This communicator:
      - Validates device capability and world size.
      - Allocates a shared symmetric buffer.
      - Chooses between 'multimem' and 'two-shot' all-reduce kernels.
      - Exposes a fast-path all_reduce() compatible with bfloat16 inputs.

    If any prerequisite is not met, the instance remains disabled and will
    decline to perform symmetric-memory all-reduce.
    """

    # Mapping: compute capability major -> supported world sizes for multimem
    # If the current (cc_major, world_size) is not listed, we fall back
    # to the two-shot path.
    _WORLD_SIZES_MULTIMEM = {
        9: [4, 6, 8],
        10: [6, 8],
    }

```
**EN:** This range introduces `TorchSymmMemCommunicator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Thin wrapper around torch-symmetric-memory collectives." In this range it coordinates distributed communication; records metrics or tracing signals.
**CN:** 这一段引入 `TorchSymmMemCommunicator`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会协调分布式通信；记录指标或追踪信号。

### Lines 54-110: Method TorchSymmMemCommunicator.__init__
```python
    def __init__(self, group: ProcessGroup, device: Union[int, str, torch.device]):
        """
        Args:
            group: Torch process group used for rendezvous and naming.
            device: Target CUDA device (index, 'cuda:X', or torch.device).
        """

        self.disabled = True

        if not torch_symm_mem_available:
            return

        if isinstance(device, int):
            device = torch.device(f"cuda:{device}")
        elif isinstance(device, str):
            device = torch.device(device)
        torch.cuda.set_device(device)
        self.dtype = torch.bfloat16
        self.device = device
        self.group = group
        self.world_size = dist.get_world_size(self.group)
        self.device_capability = torch.cuda.get_device_capability(device)[0]
        if self.device_capability < 9:
            logger.warning(
                "TorchSymmMemCommunicator: Device capability %s not supported, "
                "communicator is not available.",
                self.device_capability,
            )
            return
        if (
            self.world_size
            not in TORCH_SYMM_MEM_ALL_REDUCE_MAX_SIZES[self.device_capability]
        ):
            logger.warning(
                "TorchSymmMemCommunicator: World size %d not supported, "
                "communicator is not available.",
                self.world_size,
            )
            return
        self.max_size = TORCH_SYMM_MEM_ALL_REDUCE_MAX_SIZES[self.device_capability][
            self.world_size
        ]
        self.buffer = torch_symm_mem.empty(
            self.max_size // self.dtype.itemsize,
            device=self.device,
            dtype=self.dtype,
        )
        handle = torch_symm_mem.rendezvous(self.buffer, self.group.group_name)
        if handle.multicast_ptr == 0:
            logger.warning(
                "TorchSymmMemCommunicator: torch symmetric memory "
                "multicast operations are not supported."
            )
            self.buffer = None
            self.disabled = True
            return
        self.disabled = False
```
**EN:** This callable implements `TorchSymmMemCommunicator.__init__`. It takes `group`, `device` and mainly initializes instance state and defaults. The docstring states: "Args: group: Torch process group used for rendezvous and naming." In this range it emits logs for diagnostics; coordinates distributed communication; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TorchSymmMemCommunicator.__init__`。它接收 `group`, `device`，主要用于初始化实例状态与默认值。 在这一范围内，它会输出日志以便诊断；协调分布式通信；记录指标或追踪信号。

### Lines 112-133: Method TorchSymmMemCommunicator.should_torch_symm_mem_allreduce
```python
    def should_torch_symm_mem_allreduce(self, inp: torch.Tensor):
        """
        Fast-path eligibility check for a given tensor.

        Conditions:
          - Communicator must be enabled.
          - dtype must be bfloat16 (matches kernel + buffer dtype).
          - Total byte size must be 4-byte aligned (hardware requirement).
          - Payload must be smaller than the symmetric-memory max size.

        Returns:
            True if the symmetric-memory path can handle this tensor.
        """
        if self.disabled:
            return False
        if inp.dtype != self.dtype:
            return False
        inp_size = inp.numel() * inp.element_size()
        # enforce 4-byte alignment
        if inp_size % 4 != 0:
            return False
        return inp_size < self.max_size
```
**EN:** This callable implements `TorchSymmMemCommunicator.should_torch_symm_mem_allreduce`. It takes `inp` and mainly converts data into another representation. The docstring states: "Fast-path eligibility check for a given tensor." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TorchSymmMemCommunicator.should_torch_symm_mem_allreduce`。它接收 `inp`，主要用于将数据转换为另一种表示。 在这一范围内，它会记录指标或追踪信号。

### Lines 135-165: Method TorchSymmMemCommunicator.all_reduce
```python
    def all_reduce(
        self, inp: torch.Tensor, *, out: Optional[torch.Tensor] = None
    ) -> Optional[torch.Tensor]:
        """
        Perform an in-place sum all-reduce via torch symmetric memory.

        Args:
            inp: Input tensor on the target CUDA device (bfloat16).
            out: Optional output tensor; if omitted, a new tensor is allocated.

        Returns:
            The reduced tensor (same shape as inp), or None if disabled.

        Implementation details:
            - Stages 'inp' into the symmetric buffer.
            - Selects 'multimem' or 'two_shot' kernel based on topology.
            - Writes the result into 'out' and returns it.
        """
        if out is None:
            out = torch.empty_like(inp)
        self.buffer[: inp.numel()].copy_(inp.view(-1))
        if self.world_size in self._WORLD_SIZES_MULTIMEM[self.device_capability]:
            torch.ops.symm_mem.multimem_all_reduce_(
                self.buffer[: inp.numel()], "sum", self.group.group_name
            )
        else:
            torch.ops.symm_mem.two_shot_all_reduce_(
                self.buffer[: inp.numel()], "sum", self.group.group_name
            )
        out.copy_(self.buffer[: inp.numel()].view(out.shape))
        return out
```
**EN:** This callable implements `TorchSymmMemCommunicator.all_reduce`. It takes `inp` and mainly reduces or aggregates values. The docstring states: "Perform an in-place sum all-reduce via torch symmetric memory." In this range it coordinates distributed communication; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TorchSymmMemCommunicator.all_reduce`。它接收 `inp`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信；记录指标或追踪信号。

## Key Concepts / 关键概念
- `TorchSymmMemCommunicator`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`, `torch.distributed._symmetric_memory`
- **Internal modules / 内部模块**: `sglang.srt.distributed.device_communicators.all_reduce_utils`, `sglang.srt.utils`
