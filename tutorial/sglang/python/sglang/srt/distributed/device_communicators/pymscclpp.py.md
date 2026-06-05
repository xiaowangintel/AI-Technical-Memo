# pymscclpp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/pymscclpp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `pymscclpp`. It exposes primary entry points such as `MscclContextSelection`, `mscclpp_is_weak_contiguous`, `mscclpp_convert_to_bytes`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `pymscclpp` 的逻辑。 它对外提供的主要入口包括 `MscclContextSelection`, `mscclpp_is_weak_contiguous`, `mscclpp_convert_to_bytes`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Module imports, constants, and setup
```python
import bisect
import logging
import math
import os
from contextlib import contextmanager
from enum import IntEnum
from typing import Optional, Union

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup, ReduceOp

import sglang.srt.distributed.device_communicators.custom_all_reduce_ops as ops
from sglang.srt.utils import is_hip

logger = logging.getLogger(__name__)

_is_hip = is_hip()


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 21-23: Class MscclContextSelection
```python
class MscclContextSelection(IntEnum):
    MSCCL1SHOT1NODELL = 1
    MSCCL1SHOT2NODELL = 2
```
**EN:** This range introduces `MscclContextSelection` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `MscclContextSelection`，并定义其后续方法依赖的结构或元数据。

### Lines 26-30: Function mscclpp_is_weak_contiguous
```python
def mscclpp_is_weak_contiguous(inp: torch.Tensor):
    return inp.is_contiguous() or (
        inp.storage().nbytes() - inp.storage_offset() * inp.element_size()
        == inp.numel() * inp.element_size()
    )
```
**EN:** This callable implements `mscclpp_is_weak_contiguous`. It takes `inp` and mainly implements mscclpp is weak contiguous.
**CN:** 这一可调用对象实现了 `mscclpp_is_weak_contiguous`。它接收 `inp`，主要用于实现 mscclpp is weak contiguous 相关逻辑。

### Lines 33-71: Function mscclpp_convert_to_bytes
```python
def mscclpp_convert_to_bytes(size_str):
    """
    Converts a human-readable size string (e.g., "1MB", "2.5kb", "3 GB")
    into the equivalent number of bytes using binary units.

    Args:
        size_str (str): A string representing size with unit (KB, MB, GB).

    Returns:
        int: Number of bytes.
    """
    size_str = size_str.strip().lower()

    if not size_str:
        raise ValueError("Empty input string")

    # Extract numeric part and unit
    for i in range(len(size_str)):
        if not size_str[i].isdigit() and size_str[i] != ".":
            break
    num_str = size_str[:i]
    unit = size_str[i:].strip()

    try:
        num = float(num_str)
    except ValueError:
        raise ValueError(f"Invalid numeric value in '{size_str}'")

    # Conversion factors
    if unit == "b":
        return int(num)
    elif unit == "kb":
        return int(num * 1024)
    elif unit == "mb":
        return int(num * 1024 * 1024)
    elif unit == "gb":
        return int(num * 1024 * 1024 * 1024)
    else:
        raise ValueError(f"Unsupported unit: {unit}, support B, KB, MB, GB only")
```
**EN:** This callable implements `mscclpp_convert_to_bytes`. It takes `size_str` and mainly converts data into another representation. The docstring states: "Converts a human-readable size string (e.g., "1MB", "2.5kb", "3 GB") into the equivalent number of bytes using binary units." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `mscclpp_convert_to_bytes`。它接收 `size_str`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 74-88: Function mscclpp_bench_time
```python
def mscclpp_bench_time(func, test_niter: int = 10, warmup_niter: int = 2):
    # warmup
    for _ in range(warmup_niter):
        func()
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    torch.cuda.synchronize()
    dist.barrier()
    start_event.record()
    for _ in range(test_niter):
        func()
    end_event.record()
    end_event.synchronize()
    func_cost_us = start_event.elapsed_time(end_event) / test_niter * 1000
    return func_cost_us
```
**EN:** This callable implements `mscclpp_bench_time`. It takes `func`, `test_niter`, `warmup_niter` and mainly implements mscclpp bench time.
**CN:** 这一可调用对象实现了 `mscclpp_bench_time`。它接收 `func`, `test_niter`, `warmup_niter`，主要用于实现 mscclpp bench time 相关逻辑。

### Lines 91-97: Class PyMscclppCommunicator
```python
class PyMscclppCommunicator:
    _SUPPORTED_WORLD_SIZES = [8, 16]
    _MAX_BYTES = mscclpp_convert_to_bytes(os.getenv("SGLANG_MSCCLPP_MAX_BYTES", "1MB"))
    _SUPPORTED_DTYPE = [torch.float, torch.float16, torch.bfloat16]

    # max_bytes: max supported mscclpp allreduce size
    # in A100 mscclpp is faster than nccl only under condition of msg size smaller than1MB
```
**EN:** This range introduces `PyMscclppCommunicator` and defines the structure or metadata that its methods rely on. In this range it reads environment-driven configuration.
**CN:** 这一段引入 `PyMscclppCommunicator`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 98-157: Method PyMscclppCommunicator.__init__ (part 1/3)
```python
    def __init__(
        self,
        group: ProcessGroup,
        device: Union[int, str, torch.device],
        max_bytes=_MAX_BYTES,
    ) -> None:
        """
        Args:
            group: the process group to work on. If None, it will use the
                default process group.
            device: the device to bind the CustomAllreduce to. If None,
                it will be bind to f"cuda:{local_rank}".
        It is the caller's responsibility to make sure each communicator
        is bind to a unique device, and all communicators in this group
        are in the same node.
        """
        self._IS_CAPTURING = False
        self.disabled = True

        if not ops.IS_MSCCLPP_AR_AVAILABLE:
            # disable because of missing mscclpp library
            # e.g. in a non-cuda environment
            return

        self.group = group

        assert (
            dist.get_backend(group) != dist.Backend.NCCL
        ), "CustomAllreduce should be attached to a non-NCCL group."

        rank = dist.get_rank(group=self.group)
        world_size = dist.get_world_size(group=self.group)
        if world_size == 1:
            # No need to initialize mscclpp for single GPU case.
            return

        if world_size not in PyMscclppCommunicator._SUPPORTED_WORLD_SIZES:
            logger.warning(
                "PyMscclpp is disabled due to an unsupported world"
                " size: %d. Supported world sizes: %s. To silence this "
                "warning, specify disable_mscclpp=True explicitly.",
                world_size,
                str(PyMscclppCommunicator._SUPPORTED_WORLD_SIZES),
            )
            return

        self.ranks = torch.distributed.get_process_group_ranks(group)
        self.nranks_per_node = torch.cuda.device_count()
        # for now mscclpp with stride in the communicator is not tested
        if not (abs(self.ranks[-1] - self.ranks[0]) == world_size - 1):
            logger.warning(
                "PyMscclpp is disabled due to an unsupported group %s."
                "Please ensure all ranks in the group are consecutive."
                "To silence this warning, specify disable_mscclpp=True explicitly.",
                str(self.ranks),
            )
            return

        if isinstance(device, int):
            device = torch.device(f"cuda:{device}")
```
**EN:** This callable implements `PyMscclppCommunicator.__init__`. It takes `group`, `device`, `max_bytes` and mainly initializes instance state and defaults. The docstring states: "Args: group: the process group to work on." This chunk is part 1 of 3 for the same logical block. In this range it performs defensive checks on invalid state; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `PyMscclppCommunicator.__init__`。它接收 `group`, `device`, `max_bytes`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/3 部分。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；协调分布式通信。

### Lines 158-217: Method PyMscclppCommunicator.__init__ (part 2/3)
```python
        elif isinstance(device, str):
            device = torch.device(device)
        # now `device` is a `torch.device` object
        assert isinstance(device, torch.device)
        self.device = device

        self.max_bytes = max_bytes
        self.rank = rank
        self.world_size = world_size

        if dist.get_rank(group) == 0:
            unique_id = [ops.mscclpp_generate_unique_id()]
        else:
            unique_id = [None]
        dist.broadcast_object_list(unique_id, src=self.ranks[0], group=self.group)
        self.unique_id = unique_id[0]
        self.rank_to_node, self.rank_to_ib = list(range(world_size)), list(
            range(world_size)
        )
        for r in range(world_size):
            self.rank_to_node[r] = r // 8
            self.rank_to_ib[r] = self.rank % 8

        self._context = None
        self.context_selection = None
        self.msg_size_for_finetune = [
            2**i for i in range(10, math.floor(math.log2(self.max_bytes)) + 1)
        ]
        self.msg_size2best_config = {}
        if world_size == 8:
            self.context_selection = MscclContextSelection.MSCCL1SHOT1NODELL
        elif world_size == 16:
            self.context_selection = MscclContextSelection.MSCCL1SHOT2NODELL
        if not _is_hip:
            self.scratch = torch.empty(
                self.max_bytes * 8,
                dtype=torch.uint8,
                device=self.device,
            )
            self.put_buffer = torch.empty(
                self.max_bytes * 8 // self.nranks_per_node,
                dtype=torch.uint8,
                device=self.device,
            )
            self._context = ops.mscclpp_init_context(
                self.unique_id,
                self.rank,
                self.world_size,
                self.scratch,
                self.put_buffer,
                self.nranks_per_node,
                self.rank_to_node,
                self.rank_to_ib,
                int(self.context_selection),
            )
        else:
            raise NotImplementedError("HIP Mscclpp is not supported yet.")

        self.msg_size2best_config = {}
        self.pre_tune_config()
```
**EN:** This callable implements `PyMscclppCommunicator.__init__`. It takes `group`, `device`, `max_bytes` and mainly initializes instance state and defaults. The docstring states: "Args: group: the process group to work on." This chunk is part 2 of 3 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `PyMscclppCommunicator.__init__`。它接收 `group`, `device`, `max_bytes`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/3 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 218-228: Method PyMscclppCommunicator.__init__ (part 3/3)
```python
        if dist.get_rank(group) == 0:
            msg_size2best_config = [self.msg_size2best_config]
        else:
            msg_size2best_config = [None]
        dist.broadcast_object_list(
            msg_size2best_config, src=self.ranks[0], group=self.group
        )
        self.msg_size2best_config = msg_size2best_config[0]

        # PyMscclpp is enabled only in cuda graph
        self.disabled = True
```
**EN:** This callable implements `PyMscclppCommunicator.__init__`. It takes `group`, `device`, `max_bytes` and mainly initializes instance state and defaults. The docstring states: "Args: group: the process group to work on." This chunk is part 3 of 3 for the same logical block. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `PyMscclppCommunicator.__init__`。它接收 `group`, `device`, `max_bytes`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 3/3 部分。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 230-258: Method PyMscclppCommunicator.pre_tune_config
```python
    def pre_tune_config(self, dtype=torch.bfloat16) -> bool:
        logger.debug(f"start to pre-tune configs for rank {self.rank}")
        nthreads_to_try = [256, 512, 1024]
        nblocks_to_try = [21, 42, 84]
        inp_randn = torch.ones(
            self.msg_size_for_finetune[-1] // dtype.itemsize, dtype=dtype, device="cuda"
        )
        oup_randn = torch.empty_like(inp_randn)
        for msg_size in self.msg_size_for_finetune:
            mock_inp, mock_outp = (
                inp_randn[: msg_size // dtype.itemsize],
                oup_randn[: msg_size // dtype.itemsize],
            )
            best_config, best_time = None, None
            for nthreads in nthreads_to_try:
                for nblocks in nblocks_to_try:
                    cur_cost = mscclpp_bench_time(
                        lambda: ops.mscclpp_allreduce(
                            self._context, mock_inp, mock_outp, nthreads, nblocks
                        )
                    )
                    if best_time is None or cur_cost < best_time:
                        best_config = (nthreads, nblocks)
                        best_time = cur_cost
            self.msg_size2best_config[msg_size] = best_config
            if self.rank == 0:
                logger.debug(
                    f"for msg_size {msg_size}, best_config: {best_config}, best_time: {best_time}us"
                )
```
**EN:** This callable implements `PyMscclppCommunicator.pre_tune_config`. It takes `dtype` and mainly implements pre tune config. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `PyMscclppCommunicator.pre_tune_config`。它接收 `dtype`，主要用于实现 pre tune config 相关逻辑。 在这一范围内，它会输出日志以便诊断。

### Lines 260-274: Method PyMscclppCommunicator.should_mscclpp_allreduce
```python
    def should_mscclpp_allreduce(
        self, inp: torch.Tensor, op: ReduceOp = ReduceOp.SUM
    ) -> bool:
        if self.disabled or self._context is None:
            return False
        if inp.dtype not in PyMscclppCommunicator._SUPPORTED_DTYPE:
            return False
        if not mscclpp_is_weak_contiguous(inp):
            return False
        # only support sum op
        if op != ReduceOp.SUM:
            return False
        if inp.numel() * inp.element_size() > self.max_bytes:
            return False
        return True
```
**EN:** This callable implements `PyMscclppCommunicator.should_mscclpp_allreduce`. It takes `inp`, `op` and mainly reduces or aggregates values.
**CN:** 这一可调用对象实现了 `PyMscclppCommunicator.should_mscclpp_allreduce`。它接收 `inp`, `op`，主要用于归约或聚合数值。

### Lines 276-286: Method PyMscclppCommunicator.all_reduce
```python
    def all_reduce(self, tensor: torch.Tensor, op: ReduceOp = ReduceOp.SUM):
        if self._IS_CAPTURING:
            if torch.cuda.is_current_stream_capturing():
                self.graph_input_set.add((tensor.dtype, tensor.numel()))
        msg_size = tensor.numel() * tensor.itemsize
        index = bisect.bisect_left(self.msg_size_for_finetune, msg_size)
        msg_size_finetune = self.msg_size_for_finetune[index]
        nthreads, nblocks = self.msg_size2best_config[msg_size_finetune]
        result = torch.empty_like(tensor)
        ops.mscclpp_allreduce(self._context, tensor, result, nthreads, nblocks)
        return result
```
**EN:** This callable implements `PyMscclppCommunicator.all_reduce`. It takes `tensor`, `op` and mainly reduces or aggregates values. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `PyMscclppCommunicator.all_reduce`。它接收 `tensor`, `op`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 288-302: Method PyMscclppCommunicator.change_state
```python
    @contextmanager
    def change_state(
        self,
        enable: Optional[bool] = None,
    ):
        if enable is None:
            # guess a default value when not specified
            enable = self.available

        old_disable = self.disabled
        self.disabled = not enable

        yield

        self.disabled = old_disable
```
**EN:** This callable implements `PyMscclppCommunicator.change_state`. It takes `enable` and mainly implements change state.
**CN:** 这一可调用对象实现了 `PyMscclppCommunicator.change_state`。它接收 `enable`，主要用于实现 change state 相关逻辑。

## Key Concepts / 关键概念
- `MscclContextSelection`: core class or state container / 核心类或状态容器
- `mscclpp_is_weak_contiguous`: implements mscclpp is weak contiguous / 实现 mscclpp is weak contiguous 相关逻辑
- `mscclpp_convert_to_bytes`: converts data into another representation / 将数据转换为另一种表示
- `mscclpp_bench_time`: implements mscclpp bench time / 实现 mscclpp bench time 相关逻辑
- `PyMscclppCommunicator`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `bisect`, `logging`, `math`, `os`, `contextlib`, `enum`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.srt.distributed.device_communicators.custom_all_reduce_ops`, `sglang.srt.utils`
