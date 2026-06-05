# quick_all_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/quick_all_reduce.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `quick_all_reduce`. It exposes primary entry points such as `qr_rocm_arch_available`, `QuickReduceRegime`, `QuickAllReduce`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `quick_all_reduce` 的逻辑。 它对外提供的主要入口包括 `qr_rocm_arch_available`, `QuickReduceRegime`, `QuickAllReduce`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0

import logging
import os
from enum import Enum
from functools import cache
from typing import Union

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

import sglang.srt.distributed.device_communicators.custom_all_reduce_ops as ops
from sglang.srt.distributed.device_communicators.custom_all_reduce_utils import (
    is_full_nvlink,
    is_weak_contiguous,
)
from sglang.srt.distributed.parallel_state import in_the_same_node_as
from sglang.srt.utils import is_cuda, is_hip

logger = logging.getLogger(__name__)

_is_cuda = is_cuda()
_is_hip = is_hip()


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 27-38: Function qr_rocm_arch_available
```python
@cache
def qr_rocm_arch_available():
    if not _is_hip:
        return False
    try:
        props = torch.cuda.get_device_properties(0)
        gcn_arch = getattr(props, "gcnArchName", "")
        supported_archs = ["gfx94", "gfx95"]
        return any(gfx in gcn_arch for gfx in supported_archs)
    except Exception as e:
        logger.warning("Failed to determine ROCm for quick allreduce: %s", e)
        return False
```
**EN:** This callable implements `qr_rocm_arch_available` and mainly implements qr rocm arch available. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `qr_rocm_arch_available`，主要用于实现 qr rocm arch available 相关逻辑。 在这一范围内，它会输出日志以便诊断。

### Lines 41-46: Class QuickReduceRegime
```python
class QuickReduceRegime(Enum):
    FP = 0
    INT8 = 1
    INT6 = 2
    INT4 = 3
    NONE = 4
```
**EN:** This range introduces `QuickReduceRegime` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `QuickReduceRegime`，并定义其后续方法依赖的结构或元数据。

### Lines 47-51: Module-level constants and helpers
```python


MB = 1024 * 1024


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 52-66: Class QuickAllReduce
```python
class QuickAllReduce:

    _SUPPORTED_WORLD_SIZES = [2, 4, 8]
    _SUPPORTED_DTYPES = [torch.float16, torch.bfloat16]
    # The following data is based on kernel tests.
    # In this order [FP, INT8, INT6, INT4].
    _QR_MIN_SIZE = {
        (torch.float16, 2): [1 * MB, 2 * MB, 2 * MB, 1 * MB],
        (torch.float16, 4): [1 * MB, 16 * MB, 4 * MB, 2 * MB],
        (torch.float16, 8): [16 * MB, 4 * MB, 4 * MB, 2 * MB],
        (torch.bfloat16, 2): [2 * MB, 8 * MB, 8 * MB, 8 * MB],
        (torch.bfloat16, 4): [8 * MB, 64 * MB, 64 * MB, 16 * MB],
        (torch.bfloat16, 8): [16 * MB, 2048 * MB, 2048 * MB, 2048 * MB],
    }

```
**EN:** This range introduces `QuickAllReduce` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `QuickAllReduce`，并定义其后续方法依赖的结构或元数据。

### Lines 67-126: Method QuickAllReduce.__init__ (part 1/2)
```python
    def __init__(
        self, group: ProcessGroup, device: Union[int, str, torch.device]
    ) -> None:
        """
        Custom allreduce provides non-destructive acceleration and is
        available for CUDA and ROCm MI300 series.
        Custom quick allreduce leverages quantization for further
        acceleration on ROCm. It currently supports Q8, Q6, and Q4
        quantization formats and FP(float16, bfloat16).
        Quick allreduce is designed as a complement to custom allreduce.
        Its initialization requires even stricter conditions.
        Only the ROCm MI300 series is supported for quick allreduce at
        this time.
        Args:
            group: the process group to work on. If None, it will use the
                default process group.
            device: the device to bind the CustomAllreduce to. If None,
                it will be bind to f"cuda:{local_rank}".
        It is the caller's responsibility to make sure each communicator
        is bind to a unique device, and all communicators in this group
        are in the same node.
        """
        self.disabled = True
        if not qr_rocm_arch_available():
            logger.debug(
                "Custom quick allreduce is only supported on ROCm MI300 series."
            )
            return

        if not ops.IS_QUICK_AR_AVAILABLE:
            # disable because of missing quick reduce library
            # e.g. in a cuda environment
            logger.info(
                "Custom quick allreduce is disabled because "
                "of missing custom quick allreduce library"
            )
            return

        self.group = group
        assert (
            dist.get_backend(group) != dist.Backend.NCCL
        ), "Custom quick allreduce should be attached to a non-NCCL group."
        if not all(in_the_same_node_as(group, source_rank=0)):
            # No need to initialize custom quick allreduce for
            # multi-node case.
            logger.warning(
                "Custom quick allreduce is disabled because this "
                "process group spans across nodes."
            )
            return
        rank = dist.get_rank(group=self.group)
        world_size = dist.get_world_size(group=self.group)
        self.rank = rank
        self.world_size = world_size
        if world_size == 1:
            # No need to initialize QuickReduce for single GPU case.
            return

        if world_size not in QuickAllReduce._SUPPORTED_WORLD_SIZES:
            logger.warning(
```
**EN:** This callable implements `QuickAllReduce.__init__`. It takes `group`, `device` and mainly initializes instance state and defaults. The docstring states: "Custom allreduce provides non-destructive acceleration and is available for CUDA and ROCm MI300 series." This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `QuickAllReduce.__init__`。它接收 `group`, `device`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；协调分布式通信。

### Lines 127-167: Method QuickAllReduce.__init__ (part 2/2)
```python
                "Custom quick allreduce is disabled due to an "
                "unsupported world size: %d. Supported world sizes: %s.",
                world_size,
                str(QuickAllReduce._SUPPORTED_WORLD_SIZES),
            )
            return

        if isinstance(device, int):
            device = torch.device(f"cuda:{device}")
        elif isinstance(device, str):
            device = torch.device(device)
        assert isinstance(device, torch.device)
        self.device = device

        cuda_visible_devices = os.environ.get("CUDA_VISIBLE_DEVICES", None)
        if cuda_visible_devices:
            device_ids = list(map(int, cuda_visible_devices.split(",")))
        else:
            device_ids = list(range(torch.cuda.device_count()))
        physical_device_id = device_ids[device.index]
        tensor = torch.tensor([physical_device_id], dtype=torch.int, device="cpu")
        gather_list = [
            torch.tensor([0], dtype=torch.int, device="cpu")
            for _ in range(self.world_size)
        ]
        dist.all_gather(gather_list, tensor, group=self.group)
        physical_device_ids = [t.item() for t in gather_list]

        # test nvlink first, this will filter out most of the cases
        # where custom quick allreduce is not supported
        # this checks hardware and driver support for NVLink
        if _is_cuda or _is_hip:
            self.fully_connected = is_full_nvlink(physical_device_ids, self.world_size)
        if self.world_size > 2 and not self.fully_connected:
            logger.debug(
                "Custom quick allreduce is disabled because it's not supported "
                "on more than two PCIe-only GPUs. "
            )
            return

        self.init_quick_all_reduce()
```
**EN:** This callable implements `QuickAllReduce.__init__`. It takes `group`, `device` and mainly initializes instance state and defaults. The docstring states: "Custom allreduce provides non-destructive acceleration and is available for CUDA and ROCm MI300 series." This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `QuickAllReduce.__init__`。它接收 `group`, `device`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；协调分布式通信。

### Lines 169-211: Method QuickAllReduce.init_quick_all_reduce
```python
    def init_quick_all_reduce(self):
        # On RocM, bfloat16 kernels are slower than fp16
        # due to slower match operations
        # If environment variable is set to 1, we convert input to fp16
        self.use_fp16_kernels = int(
            os.environ.get("ROCM_QUICK_REDUCE_CAST_BF16_TO_FP16", 1)
        )
        regime_str = os.environ.get("ROCM_QUICK_REDUCE_QUANTIZATION", "NONE")
        if regime_str not in QuickReduceRegime.__members__:
            logger.warning(
                "Custom quick allreduce:",
                f"Invalid quantization level: {regime_str}. "
                "Supported levels: "
                f"{list(QuickReduceRegime.__members__.keys())}",
            )
            return

        if regime_str == "NONE":
            logger.debug(
                "Custom quick allreduce is disabled based "
                "on env variable "
                "ROCM_QUICK_REDUCE_QUANTIZATION='NONE'"
            )
            return
        self.qr_quant_level = QuickReduceRegime[regime_str]

        # TODO: If the dtype is not bfloat16 or then float16,
        # quickallreduce should not be created.

        # ROCM_QUICK_REDUCE_MAX_SIZE_BYTES_MB is specified in MB
        qr_max_size = int(os.environ.get("ROCM_QUICK_REDUCE_MAX_SIZE_BYTES_MB", 0))
        if qr_max_size > 0:
            if qr_max_size < 1:
                logger.info(
                    "You should not set a max_size smaller than 1MB, which can "
                    "lead to error or degradation to custom allreduce or rccl."
                )
            qr_max_size = qr_max_size * MB
        # If qr_max_size is None, then 2GB is used by default.
        self._ptr = ops.init_custom_qr(self.rank, self.world_size, qr_max_size)
        self.qr_max_size = qr_max_size if qr_max_size > 0 else ops.qr_max_size()
        self.create_shared_buffer()
        self.disabled = False
```
**EN:** This callable implements `QuickAllReduce.init_quick_all_reduce` and mainly reduces or aggregates values. In this range it emits logs for diagnostics; coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `QuickAllReduce.init_quick_all_reduce`，主要用于归约或聚合数值。 在这一范围内，它会输出日志以便诊断；协调分布式通信；读取环境变量驱动的配置。

### Lines 213-222: Method QuickAllReduce.create_shared_buffer
```python
    def create_shared_buffer(self):
        """
        Creates a shared buffer for quickreduce.
        Has to be called after init_custom_qr
        """
        handle = ops.qr_get_handle(self._ptr)
        world_size = dist.get_world_size(group=self.group)
        handles = [None] * world_size
        dist.all_gather_object(handles, handle, group=self.group)
        ops.qr_open_handles(self._ptr, handles)
```
**EN:** This callable implements `QuickAllReduce.create_shared_buffer` and mainly constructs new objects or contexts. The docstring states: "Creates a shared buffer for quickreduce."
**CN:** 这一可调用对象实现了 `QuickAllReduce.create_shared_buffer`，主要用于构造新的对象或上下文。

### Lines 224-246: Method QuickAllReduce.should_quick_allreduce
```python
    def should_quick_allreduce(self, inp: torch.Tensor):
        """
        Check if quickreduce is available
        """
        if self.disabled:
            return False
        if inp.dtype not in self._SUPPORTED_DTYPES:
            return False
        inp_size = inp.numel() * inp.element_size()
        # custom quick allreduce requires input byte size to be
        # multiples of 16
        if inp_size % 16 != 0:
            return False
        if not is_weak_contiguous(inp):
            return False
        dtype = inp.dtype
        if self.use_fp16_kernels:
            dtype = torch.float16
        return (
            inp_size <= self.qr_max_size
            and inp_size
            >= self._QR_MIN_SIZE[(dtype, self.world_size)][self.qr_quant_level.value]
        )
```
**EN:** This callable implements `QuickAllReduce.should_quick_allreduce`. It takes `inp` and mainly reduces or aggregates values. The docstring states: "Check if quickreduce is available"
**CN:** 这一可调用对象实现了 `QuickAllReduce.should_quick_allreduce`。它接收 `inp`，主要用于归约或聚合数值。

### Lines 248-257: Method QuickAllReduce.quick_all_reduce
```python
    def quick_all_reduce(self, inp: torch.Tensor, *, out: torch.Tensor = None):
        """Performs an out-of-place custom quick all reduce."""
        # quick allreduce doesn't require a separate graph mode,
        # as QR uses static IPC buffer.
        if out is None:
            out = torch.empty_like(inp)
        ops.qr_all_reduce(
            self._ptr, inp, out, self.qr_quant_level.value, self.use_fp16_kernels
        )
        return out
```
**EN:** This callable implements `QuickAllReduce.quick_all_reduce`. It takes `inp` and mainly reduces or aggregates values. The docstring states: "Performs an out-of-place custom quick all reduce." In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `QuickAllReduce.quick_all_reduce`。它接收 `inp`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 259-264: Method QuickAllReduce.close
```python
    def close(self):
        if not self.disabled and getattr(self, "_ptr", None):
            if ops is not None:
                ops.qr_destroy(self._ptr)
            self._ptr = 0
            self.disabled = True
```
**EN:** This callable implements `QuickAllReduce.close` and mainly implements close.
**CN:** 这一可调用对象实现了 `QuickAllReduce.close`，主要用于实现 close 相关逻辑。

### Lines 266-267: Method QuickAllReduce.__del__
```python
    def __del__(self):
        self.close()
```
**EN:** This callable implements `QuickAllReduce.__del__` and mainly implements del.
**CN:** 这一可调用对象实现了 `QuickAllReduce.__del__`，主要用于实现 del 相关逻辑。

## Key Concepts / 关键概念
- `qr_rocm_arch_available`: implements qr rocm arch available / 实现 qr rocm arch available 相关逻辑
- `QuickReduceRegime`: core class or state container / 核心类或状态容器
- `MB`: module constant or capability flag / 模块常量或能力标记
- `QuickAllReduce`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `os`, `enum`, `functools`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.srt.distributed.device_communicators.custom_all_reduce_ops`, `sglang.srt.distributed.device_communicators.custom_all_reduce_utils`, `sglang.srt.distributed.parallel_state`, `sglang.srt.utils`
