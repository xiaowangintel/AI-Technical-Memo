# parallel_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/parallel_state.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `_split_tensor_dict`, `_register_group`, and `all_reduce`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: sglang-diffusion distributed state. It takes over the control of the distributed environment from PyTorch. / 该文件属于分布式运行时层。它围绕 `_split_tensor_dict`、`_register_group` 和 `all_reduce` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 16-59: module setup and imports / 模块初始化与导入
```python
"""sglang-diffusion distributed state.

It takes over the control of the distributed environment from PyTorch.
The typical workflow is:

- call `init_distributed_environment` to initialize the distributed environment.
- call `initialize_model_parallel` or `ensure_model_parallel_initialized` to
 initialize the model parallel groups.

- any code dealing with the distributed stuff

- call `destroy_model_parallel` to destroy the model parallel groups.
- call `destroy_distributed_environment` to destroy the distributed environment.

If you only need to use the distributed environment without model parallelism,
 you can skip the model parallel initialization and destruction steps.
"""

import contextlib
import datetime
import os
import weakref
from collections import namedtuple
from collections.abc import Callable
from contextlib import contextmanager
from multiprocessing import shared_memory
from typing import Any, List, Optional
from unittest.mock import patch

import torch
import torch.distributed
from torch.distributed import ProcessGroup

import sglang.multimodal_gen.envs as envs
from sglang.multimodal_gen.runtime.distributed.utils import StatelessProcessGroup
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

from ..utils.distributed import RankGenerator
from .group_coordinator import (
    GroupCoordinator,
    PipelineGroupCoordinator,
    SequenceParallelGroupCoordinator,
    get_local_torch_device,
)
```
**EN:** This block establishes the module context and imports `contextlib`, `datetime`, `os`, `weakref`, `collections`, and `collections.abc`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `contextlib`、`datetime`、`os`、`weakref`、`collections` 和 `collections.abc`。这些依赖为后续实现提供所需符号。

### Lines 61-72: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

_WORLD: GroupCoordinator | None = None
_TP: GroupCoordinator | None = None
_SP: SequenceParallelGroupCoordinator | None = None
_PP: PipelineGroupCoordinator | None = None
_CFG: GroupCoordinator | None = None
_DP: GroupCoordinator | None = None
_DIT: ProcessGroup | None = None
_VAE: ProcessGroup | None = None

TensorMetadata = namedtuple("TensorMetadata", ["device", "dtype", "size"])
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `_WORLD`, `_TP`, `_SP`, `_PP`, and `_CFG`. The code collaborates with `init_logger`, and `namedtuple`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`_WORLD`、`_TP`、`_SP`、`_PP` 和 `_CFG` 等名称。 代码会与 `init_logger` 和 `namedtuple` 协同工作。

### Lines 75-98: `_split_tensor_dict` implementation / `_split_tensor_dict` 实现
```python
def _split_tensor_dict(
    tensor_dict: dict[str, torch.Tensor | Any],
) -> tuple[list[tuple[str, Any]], list[torch.Tensor]]:
    """Split the tensor dictionary into two parts:
    1. A list of (key, value) pairs. If the value is a tensor, it is replaced
         by its metadata.
    2. A list of tensors.
    """
    metadata_list: list[tuple[str, Any]] = []
    tensor_list: list[torch.Tensor] = []
    for key, value in tensor_dict.items():
        if isinstance(value, torch.Tensor):
            # Note: we cannot use `value.device` here,
            # because it contains not only the device type but also the device
            # index (e.g. "cuda:0"). We only need the device type.
            # receiving side will set the device index.
            device = value.device.type
            metadata_list.append(
                (key, TensorMetadata(device, value.dtype, value.size()))
            )
            tensor_list.append(value)
        else:
            metadata_list.append((key, value))
    return metadata_list, tensor_list
```
**EN:** This block defines function `_split_tensor_dict`. Split the tensor dictionary into two parts: 1. A list of (key, value) pairs. Key calls include `tensor_dict.items`, `isinstance`, `metadata_list.append`, `tensor_list.append`, and `TensorMetadata`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `tensor_dict` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_split_tensor_dict`。 它用于拆分tensor dict。 关键调用包括 `tensor_dict.items`、`isinstance`、`metadata_list.append`、`tensor_list.append` 和 `TensorMetadata`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `tensor_dict` 等参数驱动。

### Lines 101-101: supporting statements / 辅助语句
```python
_groups: dict[str, Callable[[], Optional["GroupCoordinator"]]] = {}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_groups`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_groups` 等名称。

### Lines 104-105: `_register_group` implementation / `_register_group` 实现
```python
def _register_group(group: "GroupCoordinator") -> None:
    _groups[group.unique_name] = weakref.ref(group)
```
**EN:** This block defines function `_register_group`. It registers group. Key calls include `weakref.ref`. Parameters such as `group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_register_group`。 它用于注册group。 关键调用包括 `weakref.ref`。 本段逻辑主要由 `group` 等参数驱动。

### Lines 108-113: `all_reduce` implementation / `all_reduce` 实现
```python
def all_reduce(tensor: torch.Tensor, group_name: str) -> torch.Tensor:
    assert group_name in _groups, f"Group {group_name} is not found."
    group = _groups[group_name]()
    if group is None:
        raise ValueError(f"Group {group_name} is destroyed.")
    return group._all_reduce_out_place(tensor)
```
**EN:** This block defines function `all_reduce`. It handles all reduce logic. Key calls include `_groups`, `group._all_reduce_out_place`, and `ValueError`. The implementation branches on conditions. Parameters such as `tensor`, and `group_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `all_reduce`。 它用于处理 all reduce 相关逻辑。 关键调用包括 `_groups`、`group._all_reduce_out_place` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `tensor` 和 `group_name` 等参数驱动。

### Lines 116-117: `all_reduce_fake` implementation / `all_reduce_fake` 实现
```python
def all_reduce_fake(tensor: torch.Tensor, group_name: str) -> torch.Tensor:
    return torch.empty_like(tensor)
```
**EN:** This block defines function `all_reduce_fake`. It handles all reduce fake logic. Key calls include `torch.empty_like`. Parameters such as `tensor`, and `group_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `all_reduce_fake`。 它用于处理 all reduce fake 相关逻辑。 关键调用包括 `torch.empty_like`。 本段逻辑主要由 `tensor` 和 `group_name` 等参数驱动。

### Lines 120-122: `get_world_group` implementation / `get_world_group` 实现
```python
def get_world_group() -> GroupCoordinator:
    assert _WORLD is not None, "world group is not initialized"
    return _WORLD
```
**EN:** This block defines function `get_world_group`. It retrieves world group.
**CN:** 该代码块定义了函数 `get_world_group`。 它用于获取world group。

### Lines 125-134: `init_world_group` implementation / `init_world_group` 实现
```python
def init_world_group(
    ranks: list[int], local_rank: int, backend: str
) -> GroupCoordinator:
    return GroupCoordinator(
        group_ranks=[ranks],
        local_rank=local_rank,
        torch_distributed_backend=backend,
        use_device_communicator=True,
        group_name="world",
    )
```
**EN:** This block defines function `init_world_group`. It initializes world group. Key calls include `GroupCoordinator`. Parameters such as `ranks`, `local_rank`, and `backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `init_world_group`。 它用于初始化world group。 关键调用包括 `GroupCoordinator`。 本段逻辑主要由 `ranks`、`local_rank` 和 `backend` 等参数驱动。

### Lines 137-174: `init_parallel_group_coordinator` implementation / `init_parallel_group_coordinator` 实现
```python
def init_parallel_group_coordinator(
    group_ranks: List[List[int]],
    local_rank: int,
    backend: str,
    parallel_mode: str,
    **kwargs,
) -> GroupCoordinator:
    """Return a group coordinator for the given parallel mode."""
    assert parallel_mode in [
        "data",
        "pipeline",
        "tensor",
        "sequence",
        "classifier_free_guidance",
    ], f"parallel_mode {parallel_mode} is not supported"
    if parallel_mode == "pipeline":
        return PipelineGroupCoordinator(
            group_ranks=group_ranks,
            local_rank=local_rank,
            torch_distributed_backend=backend,
            group_name="pp_group",
        )
    elif parallel_mode == "sequence":
        return SequenceParallelGroupCoordinator(
            group_ranks=group_ranks,
            local_rank=local_rank,
            torch_distributed_backend=backend,
            group_name="sp_group",
            **kwargs,
        )
    else:
        # fallback to GroupCoordinator
        return GroupCoordinator(
            group_ranks=group_ranks,
            local_rank=local_rank,
            torch_distributed_backend=backend,
            group_name="cfg_group",
        )
```
**EN:** This block defines function `init_parallel_group_coordinator`. Return a group coordinator for the given parallel mode. Key calls include `PipelineGroupCoordinator`, `SequenceParallelGroupCoordinator`, and `GroupCoordinator`. The implementation branches on conditions. Parameters such as `group_ranks`, `local_rank`, `backend`, and `parallel_mode` drive the behavior in this section.
**CN:** 该代码块定义了函数 `init_parallel_group_coordinator`。 它用于初始化parallel group coordinator。 关键调用包括 `PipelineGroupCoordinator`、`SequenceParallelGroupCoordinator` 和 `GroupCoordinator`。 实现中包含条件分支。 本段逻辑主要由 `group_ranks`、`local_rank`、`backend` 和 `parallel_mode` 等参数驱动。

### Lines 177-179: `get_tp_group` implementation / `get_tp_group` 实现
```python
def get_tp_group() -> GroupCoordinator:
    assert _TP is not None, "tensor model parallel group is not initialized"
    return _TP
```
**EN:** This block defines function `get_tp_group`. It retrieves tp group.
**CN:** 该代码块定义了函数 `get_tp_group`。 它用于获取tp group。

### Lines 182-259: `init_distributed_environment` implementation / `init_distributed_environment` 实现
```python
def init_distributed_environment(
    world_size: int = 1,
    rank: int = 0,
    distributed_init_method: str = "env://",
    local_rank: int = 0,
    backend: str | None = None,
    device_id: torch.device | None = None,
    timeout: int | None = None,
):
    # Determine the appropriate backend based on the platform
    from sglang.multimodal_gen.runtime.platforms import current_platform

    if backend is None:
        backend = current_platform.get_torch_distributed_backend_str()
        logger.info(
            "Using %s backend for %s platform", backend, current_platform.device_name
        )

    logger.debug(
        "world_size=%d rank=%d local_rank=%d "
        "distributed_init_method=%s backend=%s timeout=%s",
        world_size,
        rank,
        local_rank,
        distributed_init_method,
        backend,
        timeout,
    )
    if not torch.distributed.is_initialized():
        assert distributed_init_method is not None, (
            "distributed_init_method must be provided when initializing "
            "distributed environment"
        )

        # For MPS, MUSA, and XPU, don't pass device_id as it doesn't support device indices
        extra_args = (
            {}
            if (
                current_platform.is_mps()
                or current_platform.is_musa()
                or current_platform.is_npu()
                or current_platform.is_cpu()
                or current_platform.is_xpu()
            )
            else dict(device_id=device_id)
        )

        if timeout is not None:

            extra_args["timeout"] = datetime.timedelta(seconds=timeout)
            logger.info(f"Setting distributed timeout to {timeout} seconds")

        torch.distributed.init_process_group(
            backend=backend,
            init_method=distributed_init_method,
            world_size=world_size,
            rank=rank,
            **extra_args,
        )

    # set the local rank
    # local_rank is not available in torch ProcessGroup,
    # see https://github.com/pytorch/pytorch/issues/122816
    if local_rank == -1:
        # local rank not set, this usually happens in single-node
        # setting, where we can use rank as local rank
        if distributed_init_method == "env://":
            local_rank = envs.LOCAL_RANK
        else:
            local_rank = rank
    global _WORLD
    if _WORLD is None:
        ranks = list(range(torch.distributed.get_world_size()))
        _WORLD = init_world_group(ranks, local_rank, backend)
    else:
        assert (
            _WORLD.world_size == torch.distributed.get_world_size()
        ), "world group already initialized with a different world size"
```
**EN:** This block defines function `init_distributed_environment`. It initializes distributed environment. Key calls include `logger.debug`, `current_platform.get_torch_distributed_backend_str`, `logger.info`, `torch.distributed.is_initialized`, and `torch.distributed.init_process_group`. The implementation branches on conditions. Parameters such as `world_size`, `rank`, `distributed_init_method`, `local_rank`, and `backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `init_distributed_environment`。 它用于初始化distributed environment。 关键调用包括 `logger.debug`、`current_platform.get_torch_distributed_backend_str`、`logger.info`、`torch.distributed.is_initialized` 和 `torch.distributed.init_process_group`。 实现中包含条件分支。 本段逻辑主要由 `world_size`、`rank`、`distributed_init_method`、`local_rank` 和 `backend` 等参数驱动。

### Lines 262-264: `get_sp_group` implementation / `get_sp_group` 实现
```python
def get_sp_group() -> SequenceParallelGroupCoordinator:
    assert _SP is not None, "sequence parallel group is not initialized"
    return _SP
```
**EN:** This block defines function `get_sp_group`. It retrieves sp group.
**CN:** 该代码块定义了函数 `get_sp_group`。 它用于获取sp group。

### Lines 267-269: `get_dp_group` implementation / `get_dp_group` 实现
```python
def get_dp_group() -> GroupCoordinator:
    assert _DP is not None, "data parallel group is not initialized"
    return _DP
```
**EN:** This block defines function `get_dp_group`. It retrieves dp group.
**CN:** 该代码块定义了函数 `get_dp_group`。 它用于获取dp group。

### Lines 273-435: `initialize_model_parallel` implementation / `initialize_model_parallel` 实现
```python
def initialize_model_parallel(
    data_parallel_size: int = 1,
    classifier_free_guidance_degree: int = 1,
    sequence_parallel_degree: Optional[int] = None,
    ulysses_degree: int = 1,
    ring_degree: int = 1,
    tensor_parallel_degree: int = 1,
    pipeline_parallel_degree: int = 1,
    vae_parallel_size: int = 0,
    backend: Optional[str] = None,
) -> None:
    """
    Initialize model parallel groups.

    Arguments:
        data_parallel_size: number of data parallelism groups.
        classifier_free_guidance_degree: number of GPUs used for Classifier Free Guidance (CFG)
        sequence_parallel_degree: number of GPUs used for sequence parallelism. sequence_parallel_degree = ulysses_degree * ring_degree
        ulysses_degree: number of GPUs used for ulysses sequence parallelism.
        ring_degree: number of GPUs used for ring sequence parallelism.
        tensor_parallel_degree: number of GPUs used for tensor parallelism.
        pipeline_parallel_degree: number of GPUs used for pipeline parallelism.
        backend: distributed backend of pytorch collective comm.

    Let's say we have a total of 16 GPUs denoted by g0 ... g15 and we
    use 2 groups to parallelize the batch dim(dp), 2 groups to parallelize
    split batch caused by CFG, and 2 GPUs to parallelize sequence.

    dp_degree (2) * cfg_degree (2) * sp_degree (2) * pp_degree (2) = 16.

    The present function will create 8 data-parallel groups,
    8 CFG group, 8 pipeline-parallel group, and
    8 sequence-parallel groups:
        8 data-parallel groups:
            [g0, g8], [g1, g9], [g2, g10], [g3, g11],
            [g4, g12], [g5, g13], [g6, g14], [g7, g15]
        8 CFG-parallel groups:
            [g0, g4], [g1, g5], [g2, g6], [g3, g7],
            [g8, g12], [g9, g13], [g10, g14], [g11, g15]
        8 sequence-parallel groups:
            [g0, g1], [g2, g3], [g4, g5], [g6, g7],
            [g8, g9], [g10, g11], [g12, g13], [g14, g15]
        8 pipeline-parallel groups:
            [g0, g2], [g4, g6], [g8, g10], [g12, g14],
            [g1, g3], [g5, g7], [g9, g11], [g13, g15]
    Note that for efficiency, the caller should make sure adjacent ranks
    are on the same DGX box. For example if we are using 2 DGX-1 boxes
    with a total of 16 GPUs, rank 0 to 7 belong to the first box and
    ranks 8 to 15 belong to the second box.
    """

    if backend is None:
        from sglang.multimodal_gen.runtime.platforms import current_platform

        backend = current_platform.get_torch_distributed_backend_str()
    # Get world size and rank. Ensure some consistencies.
    assert torch.distributed.is_initialized()
    world_size: int = torch.distributed.get_world_size()
    backend = backend or torch.distributed.get_backend(get_world_group().device_group)

    dit_parallel_size = (
        data_parallel_size
        * classifier_free_guidance_degree
        * sequence_parallel_degree
        * pipeline_parallel_degree
        * tensor_parallel_degree
    )

    if world_size < dit_parallel_size:
        raise RuntimeError(
            f"world_size ({world_size}) is less than "
            f"tensor_parallel_degree ({tensor_parallel_degree}) x "
            f"pipeline_parallel_degree ({pipeline_parallel_degree}) x"
            f"sequence_parallel_degree ({sequence_parallel_degree}) x"
            f"classifier_free_guidance_degree "
            f"({classifier_free_guidance_degree}) x"
            f"data_parallel_degree ({data_parallel_size})"
        )

    rank_generator: RankGenerator = RankGenerator(
        tensor_parallel_degree,
        sequence_parallel_degree,
        pipeline_parallel_degree,
        classifier_free_guidance_degree,
        data_parallel_size,
        "tp-sp-pp-cfg-dp",
    )
    global _DP
    assert _DP is None, "data parallel group is already initialized"
    _DP = init_parallel_group_coordinator(
        group_ranks=rank_generator.get_ranks("dp"),
        local_rank=get_world_group().local_rank,
        backend=backend,
        parallel_mode="data",
    )

    global _CFG
    assert _CFG is None, "classifier_free_guidance group is already initialized"
    _CFG = init_parallel_group_coordinator(
        group_ranks=rank_generator.get_ranks("cfg"),
        local_rank=get_world_group().local_rank,
        backend=backend,
        parallel_mode="classifier_free_guidance",
    )
    global _PP
    assert _PP is None, "pipeline model parallel group is already initialized"
    _PP = init_parallel_group_coordinator(
        group_ranks=rank_generator.get_ranks("pp"),
        local_rank=get_world_group().local_rank,
        backend=backend,
        parallel_mode="pipeline",
    )

    global _SP
    assert _SP is None, "sequence parallel group is already initialized"

    try:
        from .parallel_groups import PROCESS_GROUP as _YC_PROCESS_GROUP
        from .parallel_groups import (
            set_seq_parallel_pg_by_sp_groups as _set_seq_parallel_pg_by_sp_groups,
        )
    except ImportError:
        _set_seq_parallel_pg_by_sp_groups = None

        class _DummyProcessGroup:
            ULYSSES_PG = torch.distributed.group.WORLD
            RING_PG = torch.distributed.group.WORLD

        PROCESS_GROUP = _DummyProcessGroup()
    else:
        # Build SGLang Diffusion SP sub-groups based on the true SP groups. This is
        # critical when TP>1, because SP groups may be strided in global ranks
        # (e.g., tp-sp order).
        sp_groups = rank_generator.get_ranks("sp")
        _set_seq_parallel_pg_by_sp_groups(
            sp_ulysses_degree=ulysses_degree,
            sp_ring_degree=ring_degree,
            rank=get_world_group().rank,
            sp_groups=sp_groups,
        )
        PROCESS_GROUP = _YC_PROCESS_GROUP

    _SP = init_parallel_group_coordinator(
        group_ranks=rank_generator.get_ranks("sp"),
        local_rank=get_world_group().local_rank,
        backend=backend,
        parallel_mode="sequence",
        ulysses_group=PROCESS_GROUP.ULYSSES_PG,
        ring_group=PROCESS_GROUP.RING_PG,
    )

    global _TP
    assert _TP is None, "Tensor parallel group is already initialized"
    _TP = init_parallel_group_coordinator(
        group_ranks=rank_generator.get_ranks("tp"),
        local_rank=get_world_group().local_rank,
        backend=backend,
        parallel_mode="tensor",
    )

    if vae_parallel_size > 0:
        init_vae_group(dit_parallel_size, vae_parallel_size, backend)
    init_dit_group(dit_parallel_size, backend)
```
**EN:** This block defines function `initialize_model_parallel`. Initialize model parallel groups. Arguments: data_parallel_size: number of data parallelism groups. Key calls include `torch.distributed.is_initialized`, `torch.distributed.get_world_size`, `RankGenerator`, `init_parallel_group_coordinator`, and `init_dit_group`. The implementation branches on conditions, handles exceptional paths. Parameters such as `data_parallel_size`, `classifier_free_guidance_degree`, `sequence_parallel_degree`, `ulysses_degree`, and `ring_degree` drive the behavior in this section.
**CN:** 该代码块定义了函数 `initialize_model_parallel`。 它用于初始化model parallel。 关键调用包括 `torch.distributed.is_initialized`、`torch.distributed.get_world_size`、`RankGenerator`、`init_parallel_group_coordinator` 和 `init_dit_group`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `data_parallel_size`、`classifier_free_guidance_degree`、`sequence_parallel_degree`、`ulysses_degree` 和 `ring_degree` 等参数驱动。

### Lines 438-440: `get_sp_world_size` implementation / `get_sp_world_size` 实现
```python
def get_sp_world_size() -> int:
    """Return world size for the sequence model parallel group."""
    return get_sp_group().world_size
```
**EN:** This block defines function `get_sp_world_size`. Return world size for the sequence model parallel group. Key calls include `get_sp_group`.
**CN:** 该代码块定义了函数 `get_sp_world_size`。 它用于获取sp world size。 关键调用包括 `get_sp_group`。

### Lines 443-445: `get_sp_parallel_rank` implementation / `get_sp_parallel_rank` 实现
```python
def get_sp_parallel_rank() -> int:
    """Return my rank for the sequence model parallel group."""
    return get_sp_group().rank_in_group
```
**EN:** This block defines function `get_sp_parallel_rank`. Return my rank for the sequence model parallel group. Key calls include `get_sp_group`.
**CN:** 该代码块定义了函数 `get_sp_parallel_rank`。 它用于获取sp parallel rank。 关键调用包括 `get_sp_group`。

### Lines 448-450: `get_world_size` implementation / `get_world_size` 实现
```python
def get_world_size() -> int:
    """Return world size for the world group."""
    return get_world_group().world_size
```
**EN:** This block defines function `get_world_size`. Return world size for the world group. Key calls include `get_world_group`.
**CN:** 该代码块定义了函数 `get_world_size`。 它用于获取world size。 关键调用包括 `get_world_group`。

### Lines 453-455: `get_world_rank` implementation / `get_world_rank` 实现
```python
def get_world_rank() -> int:
    """Return my rank for the world group."""
    return get_world_group().rank
```
**EN:** This block defines function `get_world_rank`. Return my rank for the world group. Key calls include `get_world_group`.
**CN:** 该代码块定义了函数 `get_world_rank`。 它用于获取world rank。 关键调用包括 `get_world_group`。

### Lines 458-460: `get_dp_world_size` implementation / `get_dp_world_size` 实现
```python
def get_dp_world_size() -> int:
    """Return world size for the data parallel group."""
    return get_dp_group().world_size
```
**EN:** This block defines function `get_dp_world_size`. Return world size for the data parallel group. Key calls include `get_dp_group`.
**CN:** 该代码块定义了函数 `get_dp_world_size`。 它用于获取dp world size。 关键调用包括 `get_dp_group`。

### Lines 463-465: `get_dp_rank` implementation / `get_dp_rank` 实现
```python
def get_dp_rank() -> int:
    """Return my rank for the data parallel group."""
    return get_dp_group().rank_in_group
```
**EN:** This block defines function `get_dp_rank`. Return my rank for the data parallel group. Key calls include `get_dp_group`.
**CN:** 该代码块定义了函数 `get_dp_rank`。 它用于获取dp rank。 关键调用包括 `get_dp_group`。

### Lines 468-525: `maybe_init_distributed_environment_and_model_parallel` implementation / `maybe_init_distributed_environment_and_model_parallel` 实现
```python
def maybe_init_distributed_environment_and_model_parallel(
    tp_size: int,
    sp_size: int,
    cfg_degree: int = 1,
    ulysses_degree: int = 1,
    ring_degree: int = 1,
    dp_size: int = 1,
    distributed_init_method: str = "env://",
    dist_timeout: int | None = None,
):
    from sglang.multimodal_gen.runtime.platforms import current_platform

    if _WORLD is not None and model_parallel_is_initialized():
        # make sure the tp and sp sizes are correct
        assert (
            get_tp_world_size() == tp_size
        ), f"You are trying to initialize model parallel groups with size {tp_size}, but they are already initialized with size {get_tp_world_size()}"
        assert (
            get_sp_world_size() == sp_size
        ), f"You are trying to initialize model parallel groups with size {sp_size}, but they are already initialized with size {get_sp_world_size()}"
        return
    local_rank = int(os.environ.get("LOCAL_RANK", 0))
    world_size = int(os.environ.get("WORLD_SIZE", 1))
    rank = int(os.environ.get("RANK", 0))
    device = get_local_torch_device()
    logger.info(
        "Initializing distributed environment with world_size=%d, device=%s, timeout=%s",
        world_size,
        device,
        dist_timeout,
        main_process_only=False,
    )

    init_distributed_environment(
        world_size=world_size,
        rank=rank,
        local_rank=local_rank,
        distributed_init_method=distributed_init_method,
        device_id=device,
        backend=current_platform.get_torch_distributed_backend_str(),
        timeout=dist_timeout,
    )
    initialize_model_parallel(
        data_parallel_size=dp_size,
        classifier_free_guidance_degree=cfg_degree,
        tensor_parallel_degree=tp_size,
        ulysses_degree=ulysses_degree,
        ring_degree=ring_degree,
        sequence_parallel_degree=sp_size,
    )

    # Only set CUDA device if we're on a CUDA platform
    if current_platform.is_cuda_alike():
        device = torch.device(f"cuda:{local_rank}")
        torch.cuda.set_device(device)
    elif current_platform.is_npu():
        device = torch.device(f"npu:{local_rank}")
        torch.npu.set_device(device)
```
**EN:** This block defines function `maybe_init_distributed_environment_and_model_parallel`. It handles maybe init distributed environment and model parallel logic. Key calls include `int`, `get_local_torch_device`, `logger.info`, `init_distributed_environment`, and `initialize_model_parallel`. The implementation branches on conditions. Parameters such as `tp_size`, `sp_size`, `cfg_degree`, `ulysses_degree`, and `ring_degree` drive the behavior in this section.
**CN:** 该代码块定义了函数 `maybe_init_distributed_environment_and_model_parallel`。 它用于处理 maybe init distributed environment and model parallel 相关逻辑。 关键调用包括 `int`、`get_local_torch_device`、`logger.info`、`init_distributed_environment` 和 `initialize_model_parallel`。 实现中包含条件分支。 本段逻辑主要由 `tp_size`、`sp_size`、`cfg_degree`、`ulysses_degree` 和 `ring_degree` 等参数驱动。

### Lines 528-536: `model_parallel_is_initialized` implementation / `model_parallel_is_initialized` 实现
```python
def model_parallel_is_initialized() -> bool:
    """Check if model parallel groups are initialized."""
    return (
        _DP is not None
        and _CFG is not None
        and _SP is not None
        and _PP is not None
        and _TP is not None
    )
```
**EN:** This block defines function `model_parallel_is_initialized`. Check if model parallel groups are initialized.
**CN:** 该代码块定义了函数 `model_parallel_is_initialized`。 它用于处理 model parallel is initialized 相关逻辑。

### Lines 539-539: supporting statements / 辅助语句
```python
_TP_STATE_PATCHED = False
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_TP_STATE_PATCHED`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_TP_STATE_PATCHED` 等名称。

### Lines 542-562: `patch_tensor_parallel_group` implementation / `patch_tensor_parallel_group` 实现
```python
@contextmanager
def patch_tensor_parallel_group(tp_group: GroupCoordinator):
    """Patch the tp group temporarily until this function ends.

    This method is for draft workers of speculative decoding to run draft model
    with different tp degree from that of target model workers.

    """
    global _TP_STATE_PATCHED
    assert not _TP_STATE_PATCHED, "Should not call when it's already patched"

    _TP_STATE_PATCHED = True
    old_tp_group = get_tp_group()
    global _TP
    _TP = tp_group
    try:
        yield
    finally:
        # restore the original state
        _TP_STATE_PATCHED = False
        _TP = old_tp_group
```
**EN:** This block defines function `patch_tensor_parallel_group`. Patch the tp group temporarily until this function ends. This method is for draft workers of speculative decoding to run draft model with different tp degree from that of target model workers. Key calls include `get_tp_group`. The implementation handles exceptional paths. Parameters such as `tp_group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `patch_tensor_parallel_group`。 它用于处理 patch tensor parallel group 相关逻辑。 关键调用包括 `get_tp_group`。 实现中处理异常路径。 本段逻辑主要由 `tp_group` 等参数驱动。

### Lines 565-567: `get_tp_world_size` implementation / `get_tp_world_size` 实现
```python
def get_tp_world_size() -> int:
    """Return world size for the tensor model parallel group."""
    return get_tp_group().world_size
```
**EN:** This block defines function `get_tp_world_size`. Return world size for the tensor model parallel group. Key calls include `get_tp_group`.
**CN:** 该代码块定义了函数 `get_tp_world_size`。 它用于获取tp world size。 关键调用包括 `get_tp_group`。

### Lines 570-572: `get_tp_rank` implementation / `get_tp_rank` 实现
```python
def get_tp_rank() -> int:
    """Return my rank for the tensor model parallel group."""
    return get_tp_group().rank_in_group
```
**EN:** This block defines function `get_tp_rank`. Return my rank for the tensor model parallel group. Key calls include `get_tp_group`.
**CN:** 该代码块定义了函数 `get_tp_rank`。 它用于获取tp rank。 关键调用包括 `get_tp_group`。

### Lines 575-581: `destroy_distributed_environment` implementation / `destroy_distributed_environment` 实现
```python
def destroy_distributed_environment() -> None:
    global _WORLD
    if _WORLD:
        _WORLD.destroy()
    _WORLD = None
    if torch.distributed.is_initialized():
        torch.distributed.destroy_process_group()
```
**EN:** This block defines function `destroy_distributed_environment`. It handles destroy distributed environment logic. Key calls include `torch.distributed.is_initialized`, `_WORLD.destroy`, and `torch.distributed.destroy_process_group`. The implementation branches on conditions.
**CN:** 该代码块定义了函数 `destroy_distributed_environment`。 它用于处理 destroy distributed environment 相关逻辑。 关键调用包括 `torch.distributed.is_initialized`、`_WORLD.destroy` 和 `torch.distributed.destroy_process_group`。 实现中包含条件分支。

### Lines 584-592: `cleanup_dist_env_and_memory` implementation / `cleanup_dist_env_and_memory` 实现
```python
def cleanup_dist_env_and_memory(shutdown_ray: bool = False):
    destroy_model_parallel()
    destroy_distributed_environment()
    with contextlib.suppress(AssertionError):
        torch.distributed.destroy_process_group()
    if shutdown_ray:
        import ray  # Lazy import Ray

        ray.shutdown()
```
**EN:** This block defines function `cleanup_dist_env_and_memory`. It handles cleanup dist env and memory logic. Key calls include `destroy_model_parallel`, `destroy_distributed_environment`, `contextlib.suppress`, `torch.distributed.destroy_process_group`, and `ray.shutdown`. The implementation branches on conditions, uses context-managed resources. Parameters such as `shutdown_ray` drive the behavior in this section.
**CN:** 该代码块定义了函数 `cleanup_dist_env_and_memory`。 它用于处理 cleanup dist env and memory 相关逻辑。 关键调用包括 `destroy_model_parallel`、`destroy_distributed_environment`、`contextlib.suppress`、`torch.distributed.destroy_process_group` 和 `ray.shutdown`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `shutdown_ray` 等参数驱动。

### Lines 595-682: `is_the_same_node_as` implementation / `is_the_same_node_as` 实现
```python
def is_the_same_node_as(
    pg: ProcessGroup | StatelessProcessGroup, source_rank: int = 0
) -> list[int]:
    """
    This is a collective operation that returns if each rank is in the same node
    as the source rank. It tests if processes are attached to the same
    memory system (shared access to shared memory).
    """
    if isinstance(pg, ProcessGroup):
        assert (
            torch.distributed.get_backend(pg) != torch.distributed.Backend.NCCL
        ), "in_the_same_node_as should be tested with a non-NCCL group."
        # local rank inside the group
        rank = torch.distributed.get_rank(group=pg)
        world_size = torch.distributed.get_world_size(group=pg)

        # global ranks of the processes in the group
        ranks = torch.distributed.get_process_group_ranks(pg)
    else:
        rank = pg.rank
        world_size = pg.world_size
        ranks = list(range(world_size))

    # local tensor in each process to store the result
    is_in_the_same_node = torch.tensor([0] * world_size, dtype=torch.int32)

    magic_message = b"magic_message"
    shm = None

    try:
        with contextlib.suppress(OSError):
            if rank == source_rank:
                # create a shared memory segment
                shm = shared_memory.SharedMemory(create=True, size=128)
                shm.buf[: len(magic_message)] = magic_message
                if isinstance(pg, ProcessGroup):
                    torch.distributed.broadcast_object_list(
                        [shm.name], src=ranks[source_rank], group=pg
                    )
                else:
                    pg.broadcast_obj(shm.name, src=source_rank)
                is_in_the_same_node[rank] = 1
            else:
                # try to open the shared memory segment
                if isinstance(pg, ProcessGroup):
                    recv = [None]
                    torch.distributed.broadcast_object_list(
                        recv, src=ranks[source_rank], group=pg
                    )
                    name = recv[0]
                else:
                    name = pg.broadcast_obj(None, src=source_rank)
                # fix to https://stackoverflow.com/q/62748654/9191338
                # Python incorrectly tracks shared memory even if it is not
                # created by the process. The following patch is a workaround.
                with patch(
                    "multiprocessing.resource_tracker.register",
                    lambda *args, **kwargs: None,
                ):
                    shm = shared_memory.SharedMemory(name=name)
                if shm.buf[: len(magic_message)] == magic_message:
                    is_in_the_same_node[rank] = 1
    except Exception as e:
        logger.error("Error ignored in is_in_the_same_node: %s", e)
    finally:
        if shm:
            shm.close()

    if isinstance(pg, ProcessGroup):
        torch.distributed.barrier(group=pg)
    else:
        pg.barrier()

    # clean up the shared memory segment
    with contextlib.suppress(OSError):
        if rank == source_rank and shm:
            shm.unlink()

    if isinstance(pg, ProcessGroup):
        torch.distributed.all_reduce(is_in_the_same_node, group=pg)
        aggregated_data = is_in_the_same_node
    else:
        aggregated_data = torch.zeros_like(is_in_the_same_node)
        for i in range(world_size):
            rank_data = pg.broadcast_obj(is_in_the_same_node, src=i)
            aggregated_data += rank_data

    return [x == 1 for x in aggregated_data.tolist()]
```
**EN:** This block defines function `is_the_same_node_as`. This is a collective operation that returns if each rank is in the same node as the source rank. It tests if processes are attached to the same memory system (shared access to shared memory). Key calls include `isinstance`, `torch.tensor`, `torch.distributed.get_rank`, `torch.distributed.get_world_size`, and `torch.distributed.get_process_group_ranks`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths, uses context-managed resources. Parameters such as `pg`, and `source_rank` drive the behavior in this section.
**CN:** 该代码块定义了函数 `is_the_same_node_as`。 它用于处理 is the same node as 相关逻辑。 关键调用包括 `isinstance`、`torch.tensor`、`torch.distributed.get_rank`、`torch.distributed.get_world_size` 和 `torch.distributed.get_process_group_ranks`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `pg` 和 `source_rank` 等参数驱动。

### Lines 685-687: `get_tensor_model_parallel_world_size` implementation / `get_tensor_model_parallel_world_size` 实现
```python
def get_tensor_model_parallel_world_size() -> int:
    """Return world size for the tensor model parallel group."""
    return get_tp_world_size()
```
**EN:** This block defines function `get_tensor_model_parallel_world_size`. Return world size for the tensor model parallel group. Key calls include `get_tp_world_size`.
**CN:** 该代码块定义了函数 `get_tensor_model_parallel_world_size`。 它用于获取tensor model parallel world size。 关键调用包括 `get_tp_world_size`。

### Lines 690-692: `get_tensor_model_parallel_rank` implementation / `get_tensor_model_parallel_rank` 实现
```python
def get_tensor_model_parallel_rank() -> int:
    """Return my rank for the tensor model parallel group."""
    return get_tp_rank()
```
**EN:** This block defines function `get_tensor_model_parallel_rank`. Return my rank for the tensor model parallel group. Key calls include `get_tp_rank`.
**CN:** 该代码块定义了函数 `get_tensor_model_parallel_rank`。 它用于获取tensor model parallel rank。 关键调用包括 `get_tp_rank`。

### Lines 695-697: `get_sequence_parallel_world_size` implementation / `get_sequence_parallel_world_size` 实现
```python
def get_sequence_parallel_world_size() -> int:
    """Return world size for the sequence parallel group."""
    return get_sp_world_size()
```
**EN:** This block defines function `get_sequence_parallel_world_size`. Return world size for the sequence parallel group. Key calls include `get_sp_world_size`.
**CN:** 该代码块定义了函数 `get_sequence_parallel_world_size`。 它用于获取sequence parallel world size。 关键调用包括 `get_sp_world_size`。

### Lines 700-702: `get_sequence_parallel_rank` implementation / `get_sequence_parallel_rank` 实现
```python
def get_sequence_parallel_rank() -> int:
    """Return my rank for the sequence parallel group."""
    return get_sp_parallel_rank()
```
**EN:** This block defines function `get_sequence_parallel_rank`. Return my rank for the sequence parallel group. Key calls include `get_sp_parallel_rank`.
**CN:** 该代码块定义了函数 `get_sequence_parallel_rank`。 它用于获取sequence parallel rank。 关键调用包括 `get_sp_parallel_rank`。

### Lines 705-706: `get_ulysses_parallel_world_size` implementation / `get_ulysses_parallel_world_size` 实现
```python
def get_ulysses_parallel_world_size() -> int:
    return get_sp_group().ulysses_world_size
```
**EN:** This block defines function `get_ulysses_parallel_world_size`. It retrieves ulysses parallel world size. Key calls include `get_sp_group`.
**CN:** 该代码块定义了函数 `get_ulysses_parallel_world_size`。 它用于获取ulysses parallel world size。 关键调用包括 `get_sp_group`。

### Lines 709-710: `get_ulysses_parallel_rank` implementation / `get_ulysses_parallel_rank` 实现
```python
def get_ulysses_parallel_rank() -> int:
    return get_sp_group().ulysses_rank
```
**EN:** This block defines function `get_ulysses_parallel_rank`. It retrieves ulysses parallel rank. Key calls include `get_sp_group`.
**CN:** 该代码块定义了函数 `get_ulysses_parallel_rank`。 它用于获取ulysses parallel rank。 关键调用包括 `get_sp_group`。

### Lines 713-714: `get_ring_parallel_world_size` implementation / `get_ring_parallel_world_size` 实现
```python
def get_ring_parallel_world_size() -> int:
    return get_sp_group().ring_world_size
```
**EN:** This block defines function `get_ring_parallel_world_size`. It retrieves ring parallel world size. Key calls include `get_sp_group`.
**CN:** 该代码块定义了函数 `get_ring_parallel_world_size`。 它用于获取ring parallel world size。 关键调用包括 `get_sp_group`。

### Lines 717-718: `get_ring_parallel_rank` implementation / `get_ring_parallel_rank` 实现
```python
def get_ring_parallel_rank() -> int:
    return get_sp_group().ring_rank
```
**EN:** This block defines function `get_ring_parallel_rank`. It retrieves ring parallel rank. Key calls include `get_sp_group`.
**CN:** 该代码块定义了函数 `get_ring_parallel_rank`。 它用于获取ring parallel rank。 关键调用包括 `get_sp_group`。

### Lines 722-724: `get_pp_group` implementation / `get_pp_group` 实现
```python
def get_pp_group() -> PipelineGroupCoordinator:
    assert _PP is not None, "pipeline model parallel group is not initialized"
    return _PP
```
**EN:** This block defines function `get_pp_group`. It retrieves pp group.
**CN:** 该代码块定义了函数 `get_pp_group`。 它用于获取pp group。

### Lines 727-729: `get_pipeline_parallel_world_size` implementation / `get_pipeline_parallel_world_size` 实现
```python
def get_pipeline_parallel_world_size() -> int:
    """Return world size for the pipeline model parallel group."""
    return get_pp_group().world_size
```
**EN:** This block defines function `get_pipeline_parallel_world_size`. Return world size for the pipeline model parallel group. Key calls include `get_pp_group`.
**CN:** 该代码块定义了函数 `get_pipeline_parallel_world_size`。 它用于获取pipeline parallel world size。 关键调用包括 `get_pp_group`。

### Lines 732-734: `get_pipeline_parallel_rank` implementation / `get_pipeline_parallel_rank` 实现
```python
def get_pipeline_parallel_rank() -> int:
    """Return my rank for the pipeline model parallel group."""
    return get_pp_group().rank_in_group
```
**EN:** This block defines function `get_pipeline_parallel_rank`. Return my rank for the pipeline model parallel group. Key calls include `get_pp_group`.
**CN:** 该代码块定义了函数 `get_pipeline_parallel_rank`。 它用于获取pipeline parallel rank。 关键调用包括 `get_pp_group`。

### Lines 737-739: `is_pipeline_first_stage` implementation / `is_pipeline_first_stage` 实现
```python
def is_pipeline_first_stage() -> bool:
    """Return True if in the first pipeline model parallel stage, False otherwise."""
    return get_pipeline_parallel_rank() == 0
```
**EN:** This block defines function `is_pipeline_first_stage`. Return True if in the first pipeline model parallel stage, False otherwise. Key calls include `get_pipeline_parallel_rank`.
**CN:** 该代码块定义了函数 `is_pipeline_first_stage`。 它用于处理 is pipeline first stage 相关逻辑。 关键调用包括 `get_pipeline_parallel_rank`。

### Lines 742-744: `is_pipeline_last_stage` implementation / `is_pipeline_last_stage` 实现
```python
def is_pipeline_last_stage() -> bool:
    """Return True if in the last pipeline model parallel stage, False otherwise."""
    return get_pipeline_parallel_rank() == (get_pipeline_parallel_world_size() - 1)
```
**EN:** This block defines function `is_pipeline_last_stage`. Return True if in the last pipeline model parallel stage, False otherwise. Key calls include `get_pipeline_parallel_rank`, and `get_pipeline_parallel_world_size`.
**CN:** 该代码块定义了函数 `is_pipeline_last_stage`。 它用于处理 is pipeline last stage 相关逻辑。 关键调用包括 `get_pipeline_parallel_rank` 和 `get_pipeline_parallel_world_size`。

### Lines 748-752: `get_cfg_group` implementation / `get_cfg_group` 实现
```python
def get_cfg_group() -> GroupCoordinator:
    assert (
        _CFG is not None
    ), "classifier_free_guidance parallel group is not initialized"
    return _CFG
```
**EN:** This block defines function `get_cfg_group`. It retrieves cfg group.
**CN:** 该代码块定义了函数 `get_cfg_group`。 它用于获取cfg group。

### Lines 755-757: `get_classifier_free_guidance_world_size` implementation / `get_classifier_free_guidance_world_size` 实现
```python
def get_classifier_free_guidance_world_size() -> int:
    """Return world size for the classifier_free_guidance parallel group."""
    return get_cfg_group().world_size
```
**EN:** This block defines function `get_classifier_free_guidance_world_size`. Return world size for the classifier_free_guidance parallel group. Key calls include `get_cfg_group`.
**CN:** 该代码块定义了函数 `get_classifier_free_guidance_world_size`。 它用于获取classifier free guidance world size。 关键调用包括 `get_cfg_group`。

### Lines 760-762: `get_classifier_free_guidance_rank` implementation / `get_classifier_free_guidance_rank` 实现
```python
def get_classifier_free_guidance_rank() -> int:
    """Return my rank for the classifier_free_guidance parallel group."""
    return get_cfg_group().rank_in_group
```
**EN:** This block defines function `get_classifier_free_guidance_rank`. Return my rank for the classifier_free_guidance parallel group. Key calls include `get_cfg_group`.
**CN:** 该代码块定义了函数 `get_classifier_free_guidance_rank`。 它用于获取classifier free guidance rank。 关键调用包括 `get_cfg_group`。

### Lines 765-767: `get_data_parallel_world_size` implementation / `get_data_parallel_world_size` 实现
```python
def get_data_parallel_world_size() -> int:
    """Return world size for the data parallel group."""
    return get_dp_world_size()
```
**EN:** This block defines function `get_data_parallel_world_size`. Return world size for the data parallel group. Key calls include `get_dp_world_size`.
**CN:** 该代码块定义了函数 `get_data_parallel_world_size`。 它用于获取data parallel world size。 关键调用包括 `get_dp_world_size`。

### Lines 770-772: `get_data_parallel_rank` implementation / `get_data_parallel_rank` 实现
```python
def get_data_parallel_rank() -> int:
    """Return my rank for the data parallel group."""
    return get_dp_rank()
```
**EN:** This block defines function `get_data_parallel_rank`. Return my rank for the data parallel group. Key calls include `get_dp_rank`.
**CN:** 该代码块定义了函数 `get_data_parallel_rank`。 它用于获取data parallel rank。 关键调用包括 `get_dp_rank`。

### Lines 775-782: `is_dp_last_group` implementation / `is_dp_last_group` 实现
```python
def is_dp_last_group() -> bool:
    """Return True if in the last data parallel group, False otherwise."""
    return (
        get_sequence_parallel_rank() == (get_sequence_parallel_world_size() - 1)
        and get_classifier_free_guidance_rank()
        == (get_classifier_free_guidance_world_size() - 1)
        and get_pipeline_parallel_rank() == (get_pipeline_parallel_world_size() - 1)
    )
```
**EN:** This block defines function `is_dp_last_group`. Return True if in the last data parallel group, False otherwise. Key calls include `get_sequence_parallel_rank`, `get_classifier_free_guidance_rank`, `get_pipeline_parallel_rank`, `get_sequence_parallel_world_size`, and `get_classifier_free_guidance_world_size`.
**CN:** 该代码块定义了函数 `is_dp_last_group`。 它用于处理 is dp last group 相关逻辑。 关键调用包括 `get_sequence_parallel_rank`、`get_classifier_free_guidance_rank`、`get_pipeline_parallel_rank`、`get_sequence_parallel_world_size` 和 `get_classifier_free_guidance_world_size`。

### Lines 785-793: `get_dit_world_size` implementation / `get_dit_world_size` 实现
```python
def get_dit_world_size() -> int:
    """Return world size for the DiT model (excluding VAE)."""
    return (
        get_data_parallel_world_size()
        * get_classifier_free_guidance_world_size()
        * get_sequence_parallel_world_size()
        * get_pipeline_parallel_world_size()
        * get_tensor_model_parallel_world_size()
    )
```
**EN:** This block defines function `get_dit_world_size`. Return world size for the DiT model (excluding VAE). Key calls include `get_tensor_model_parallel_world_size`, `get_pipeline_parallel_world_size`, `get_sequence_parallel_world_size`, `get_data_parallel_world_size`, and `get_classifier_free_guidance_world_size`.
**CN:** 该代码块定义了函数 `get_dit_world_size`。 它用于获取dit world size。 关键调用包括 `get_tensor_model_parallel_world_size`、`get_pipeline_parallel_world_size`、`get_sequence_parallel_world_size`、`get_data_parallel_world_size` 和 `get_classifier_free_guidance_world_size`。

### Lines 796-798: `get_vae_parallel_group` implementation / `get_vae_parallel_group` 实现
```python
def get_vae_parallel_group() -> ProcessGroup:
    assert _VAE is not None, "VAE parallel group is not initialized"
    return _VAE
```
**EN:** This block defines function `get_vae_parallel_group`. It retrieves vae parallel group.
**CN:** 该代码块定义了函数 `get_vae_parallel_group`。 它用于获取vae parallel group。

### Lines 801-803: `get_vae_parallel_world_size` implementation / `get_vae_parallel_world_size` 实现
```python
def get_vae_parallel_world_size() -> int:
    """Return world size for the VAE parallel group."""
    return torch.distributed.get_world_size(group=get_vae_parallel_group())
```
**EN:** This block defines function `get_vae_parallel_world_size`. Return world size for the VAE parallel group. Key calls include `torch.distributed.get_world_size`, and `get_vae_parallel_group`.
**CN:** 该代码块定义了函数 `get_vae_parallel_world_size`。 它用于获取vae parallel world size。 关键调用包括 `torch.distributed.get_world_size` 和 `get_vae_parallel_group`。

### Lines 806-808: `get_vae_parallel_rank` implementation / `get_vae_parallel_rank` 实现
```python
def get_vae_parallel_rank() -> int:
    """Return my rank for the VAE parallel group."""
    return torch.distributed.get_rank(group=get_vae_parallel_group())
```
**EN:** This block defines function `get_vae_parallel_rank`. Return my rank for the VAE parallel group. Key calls include `torch.distributed.get_rank`, and `get_vae_parallel_group`.
**CN:** 该代码块定义了函数 `get_vae_parallel_rank`。 它用于获取vae parallel rank。 关键调用包括 `torch.distributed.get_rank` 和 `get_vae_parallel_group`。

### Lines 811-819: `init_dit_group` implementation / `init_dit_group` 实现
```python
def init_dit_group(
    dit_parallel_size: int,
    backend: str,
) -> None:
    global _DIT
    assert _DIT is None, "DIT group is already initialized"
    _DIT = torch.distributed.new_group(
        ranks=list(range(dit_parallel_size)), backend=backend
    )
```
**EN:** This block defines function `init_dit_group`. It initializes dit group. Key calls include `torch.distributed.new_group`, `list`, and `range`. Parameters such as `dit_parallel_size`, and `backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `init_dit_group`。 它用于初始化dit group。 关键调用包括 `torch.distributed.new_group`、`list` 和 `range`。 本段逻辑主要由 `dit_parallel_size` 和 `backend` 等参数驱动。

### Lines 822-824: `get_dit_group` implementation / `get_dit_group` 实现
```python
def get_dit_group() -> ProcessGroup:
    assert _DIT is not None, "DIT group is not initialized"
    return _DIT
```
**EN:** This block defines function `get_dit_group`. It retrieves dit group.
**CN:** 该代码块定义了函数 `get_dit_group`。 它用于获取dit group。

### Lines 827-836: `init_vae_group` implementation / `init_vae_group` 实现
```python
def init_vae_group(
    dit_parallel_size: int,
    vae_parallel_size: int,
    backend: str,
):
    # Initialize VAE group first
    global _VAE
    assert _VAE is None, "VAE parallel group is already initialized"
    vae_ranks = list(range(dit_parallel_size, dit_parallel_size + vae_parallel_size))
    _VAE = torch.distributed.new_group(ranks=vae_ranks, backend=backend)
```
**EN:** This block defines function `init_vae_group`. It initializes vae group. Key calls include `list`, `torch.distributed.new_group`, and `range`. Parameters such as `dit_parallel_size`, `vae_parallel_size`, and `backend` drive the behavior in this section.
**CN:** 该代码块定义了函数 `init_vae_group`。 它用于初始化vae group。 关键调用包括 `list`、`torch.distributed.new_group` 和 `range`。 本段逻辑主要由 `dit_parallel_size`、`vae_parallel_size` 和 `backend` 等参数驱动。

### Lines 839-851: `destroy_model_parallel` implementation / `destroy_model_parallel` 实现
```python
def destroy_model_parallel() -> None:
    """Set the groups to none and destroy them."""
    global _TP, _SP, _DP, _CFG, _PP, _DIT, _VAE

    for group in (_TP, _SP, _DP, _CFG, _PP):
        if group is not None:
            group.destroy()

    for group in (_DIT, _VAE):
        if group is not None:
            torch.distributed.destroy_process_group(group)

    _TP, _SP, _DP, _CFG, _PP, _DIT, _VAE = (None,) * 7
```
**EN:** This block defines function `destroy_model_parallel`. Set the groups to none and destroy them. Key calls include `group.destroy`, and `torch.distributed.destroy_process_group`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了函数 `destroy_model_parallel`。 它用于处理 destroy model parallel 相关逻辑。 关键调用包括 `group.destroy` 和 `torch.distributed.destroy_process_group`。 实现中包含条件分支，会遍历集合或步骤。

## Key Concepts / 关键概念
- `_split_tensor_dict`: Split the tensor dictionary into two parts: 1. / 顶层函数，用于拆分tensor dict。
- `_register_group`: Top-level function that registers group. / 顶层函数，用于注册group。
- `all_reduce`: Top-level function that handles all reduce logic. / 顶层函数，用于处理 all reduce 相关逻辑。
- `all_reduce_fake`: Top-level function that handles all reduce fake logic. / 顶层函数，用于处理 all reduce fake 相关逻辑。
- `get_world_group`: Top-level function that retrieves world group. / 顶层函数，用于获取world group。
- `init_world_group`: Top-level function that initializes world group. / 顶层函数，用于初始化world group。
- `init_parallel_group_coordinator`: Return a group coordinator for the given parallel mode. / 顶层函数，用于初始化parallel group coordinator。
- `get_tp_group`: Top-level function that retrieves tp group. / 顶层函数，用于获取tp group。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `datetime`, `os`, `weakref`, `collections`, `collections.abc`, `multiprocessing`, `typing`, `unittest.mock`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `ray`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.envs`, `sglang.multimodal_gen.runtime.distributed.utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `..utils.distributed`, `.group_coordinator`, `sglang.multimodal_gen.runtime.platforms`, `.parallel_groups`

- **Total lines / 总行数**: 851
