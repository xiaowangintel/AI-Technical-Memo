# custom_all_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/device_communicators/custom_all_reduce.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements distributed device-communication helpers or backend adapters. / 实现分布式设备通信辅助逻辑或后端适配器。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
from contextlib import contextmanager
from typing import cast

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

import vllm.envs as envs
from vllm import _custom_ops as ops
from vllm.distributed.device_communicators.all_reduce_utils import (
    CUSTOM_ALL_REDUCE_MAX_SIZES,
    gpu_p2p_access_check,
)
from vllm.distributed.parallel_state import in_the_same_node_as
from vllm.logger import init_logger
from vllm.platforms import current_platform
```
**EN:** This block imports `contextlib`, `typing`, `torch`, `torch.distributed`, `vllm.envs`, `vllm` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `contextlib`, `typing`, `torch`, `torch.distributed`, `vllm.envs`, `vllm`，为后续实现准备运行时、类型与辅助 API。

### Guarded block / 保护代码块
```python
try:
    ops.meta_size()
    custom_ar = True
except Exception:
    # For CPUs
    custom_ar = False
```
**EN:** This guarded block wraps optional setup or compatibility logic in a top-level try/except.
**CN:** 该保护代码块使用顶层 try/except 封装可选初始化或兼容性逻辑。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Function `_can_p2p` / 函数 `_can_p2p`
```python
def _can_p2p(rank: int, world_size: int) -> bool:
    for i in range(world_size):
        if i == rank:
            continue
        if envs.VLLM_SKIP_P2P_CHECK:
            logger.debug("Skipping P2P check and trusting the driver's P2P report.")
            return torch.cuda.can_device_access_peer(rank, i)
        if not gpu_p2p_access_check(rank, i):
            return False
    return True
```
**EN:** `_can_p2p` implements a focused helper routine for this module. It primarily works with arguments like `rank`, `world_size`. Key calls include `range`, `logger.debug`, `torch.cuda.can_device_access_peer`.
**CN:** `_can_p2p` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `rank`, `world_size` 这样的参数。 关键调用包括 `range`, `logger.debug`, `torch.cuda.can_device_access_peer`。

### Function `is_weak_contiguous` / 函数 `is_weak_contiguous`
```python
def is_weak_contiguous(inp: torch.Tensor):
    return inp.is_contiguous() or (
        inp.storage().nbytes() - inp.storage_offset() * inp.element_size()
        == inp.numel() * inp.element_size()
    )
```
**EN:** `is_weak_contiguous` checks a boolean property or state for this module. It primarily works with arguments like `inp`. Key calls include `inp.is_contiguous`, `inp.storage().nbytes`, `inp.numel`.
**CN:** `is_weak_contiguous` 负责检查布尔属性或状态。 它主要处理诸如 `inp` 这样的参数。 关键调用包括 `inp.is_contiguous`, `inp.storage().nbytes`, `inp.numel`。

### Class `CustomAllreduce` / 类 `CustomAllreduce`
```python
class CustomAllreduce:
    _SUPPORTED_WORLD_SIZES = [2, 4, 6, 8]

    # max_size: max supported allreduce size
    def __init__(
        self,
        group: ProcessGroup,
        device: int | str | torch.device,
        max_size=8192 * 1024,
        symm_mem_enabled=False,
    ) -> None:
        """
        Args:
            group: the process group to work on. If None, it will use the
                default process group.
            device: the device to bind the CustomAllreduce to. If None,
                it will be bound to f"cuda:{local_rank}".
        It is the caller's responsibility to make sure each communicator
        is bind to a unique device, and all communicators in this group
        are in the same node.
        """
        self._IS_CAPTURING = False
        self.disabled = True

        if not custom_ar:
            # disable because of missing custom allreduce library
            # e.g. in a non-GPU environment
            logger.info(
                "Custom allreduce is disabled because "
                "of missing custom allreduce library"
            )
            return

        self.group = group

        assert dist.get_backend(group) != dist.Backend.NCCL, (
            "CustomAllreduce should be attached to a non-NCCL group."
        )

        if not all(in_the_same_node_as(group, source_rank=0)):
            # No need to initialize custom allreduce for multi-node case.
            logger.warning(
                "Custom allreduce is disabled because this process group"
                " spans across nodes."
            )
# ... truncated for analysis ...
    @staticmethod
    def free_shared_buffer(
        pointers: list[int],
        group: ProcessGroup | None = None,
        rank: int | None = None,
    ) -> None:
        if rank is None:
            rank = dist.get_rank(group=group)
        if ops is not None:
            ops.free_shared_buffer(pointers[rank])
```
**EN:** Declares `CustomAllreduce`, a class. Key methods include `__init__`, `capture`, `register_graph_buffers`, `should_custom_ar`, `all_reduce`.
**CN:** 声明 `CustomAllreduce`，它是一个类。 关键方法包括 `__init__`, `capture`, `register_graph_buffers`, `should_custom_ar`, `all_reduce`。

## Key Concepts / 关键概念
- Collective communication backends / collective 通信后端
- `_can_p2p`: module-level helper or API entry / `_can_p2p`：模块级辅助函数或 API 入口
- `is_weak_contiguous`: module-level helper or API entry / `is_weak_contiguous`：模块级辅助函数或 API 入口
- `CustomAllreduce`: class interface or data carrier / `CustomAllreduce`：类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `vllm.envs`, `vllm`, `vllm.distributed.device_communicators.all_reduce_utils`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.platforms`
