# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared distributed utilities used across communication and runtime code. / 提供通信与运行时代码共用的分布式工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import dataclasses
import functools
import os
import pickle
import socket
import sys
import time
import uuid
from collections import deque
from collections.abc import Sequence
from datetime import timedelta
from typing import Any

import torch
from torch.distributed import ProcessGroup, Store, TCPStore
from torch.distributed.distributed_c10d import (
    Backend,
    PrefixStore,
    _get_default_timeout,
    _unregister_process_group,
)
from torch.distributed.rendezvous import rendezvous

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.utils.network_utils import get_tcp_uri
from vllm.utils.system_utils import suppress_stdout
```
**EN:** This block imports `dataclasses`, `functools`, `os`, `pickle`, `socket`, `sys` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `dataclasses`, `functools`, `os`, `pickle`, `socket`, `sys`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)

# We prefer to use os.sched_yield as it results in tighter polling loops,
# measured to be around 3e-7 seconds. However on earlier versions of Python
# os.sched_yield() does not release the GIL, so we fall back to time.sleep(0)
USE_SCHED_YIELD = (sys.version_info[:3] >= (3, 11, 1)) or (
    sys.version_info[:2] == (3, 10) and sys.version_info[2] >= 8
)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, `USE_SCHED_YIELD`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`, `USE_SCHED_YIELD`，供后续代码复用。

### Function `sched_yield` / 函数 `sched_yield`
```python
def sched_yield():
    if USE_SCHED_YIELD:
        os.sched_yield()
    else:
        time.sleep(0)
```
**EN:** `sched_yield` implements a focused helper routine for this module. Key calls include `os.sched_yield`, `time.sleep`.
**CN:** `sched_yield` 实现了一个面向当前模块的辅助例程。 关键调用包括 `os.sched_yield`, `time.sleep`。

### Function `ensure_divisibility` / 函数 `ensure_divisibility`
```python
def ensure_divisibility(numerator, denominator):
    """Ensure that numerator is divisible by the denominator."""
    assert numerator % denominator == 0, "{} is not divisible by {}".format(
        numerator, denominator
    )
```
**EN:** `ensure_divisibility` ensures a precondition or initialized state for this module. The docstring frames it as: Ensure that numerator is divisible by the denominator. It primarily works with arguments like `numerator`, `denominator`. Key calls include `format`.
**CN:** `ensure_divisibility` 负责确保前置条件或初始化状态成立。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `numerator`, `denominator` 这样的参数。 关键调用包括 `format`。

### Function `divide` / 函数 `divide`
```python
def divide(numerator, denominator):
    """Ensure that numerator is divisible by the denominator and return
    the division value."""
    ensure_divisibility(numerator, denominator)
    return numerator // denominator
```
**EN:** `divide` implements a focused helper routine for this module. The docstring frames it as: Ensure that numerator is divisible by the denominator and return the division value. It primarily works with arguments like `numerator`, `denominator`. Key calls include `ensure_divisibility`.
**CN:** `divide` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `numerator`, `denominator` 这样的参数。 关键调用包括 `ensure_divisibility`。

### Function `split_tensor_along_last_dim` / 函数 `split_tensor_along_last_dim`
```python
def split_tensor_along_last_dim(
    tensor: torch.Tensor,
    num_partitions: int,
    contiguous_split_chunks: bool = False,
) -> Sequence[torch.Tensor]:
    """Split a tensor along its last dimension.

    Arguments:
        tensor: input tensor.
        num_partitions: number of partitions to split the tensor
        contiguous_split_chunks: If True, make each chunk contiguous
                                 in memory.

    Returns:
        A list of Tensors
    """
    # Get the size and dimension.
    last_dim = tensor.dim() - 1
    last_dim_size = divide(tensor.size()[last_dim], num_partitions)
    # Split.
    tensor_list = torch.split(tensor, last_dim_size, dim=last_dim)
    # NOTE: torch.split does not create contiguous tensors by default.
    if contiguous_split_chunks:
        return tuple(chunk.contiguous() for chunk in tensor_list)

    return tensor_list
```
**EN:** `split_tensor_along_last_dim` implements a focused helper routine for this module. The docstring frames it as: Split a tensor along its last dimension. It primarily works with arguments like `tensor`, `num_partitions`, `contiguous_split_chunks`. Key calls include `divide`, `torch.split`, `tensor.dim`.
**CN:** `split_tensor_along_last_dim` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `tensor`, `num_partitions`, `contiguous_split_chunks` 这样的参数。 关键调用包括 `divide`, `torch.split`, `tensor.dim`。

### Function `get_pp_indices` / 函数 `get_pp_indices`
```python
def get_pp_indices(
    num_hidden_layers: int, pp_rank: int, pp_size: int
) -> tuple[int, int]:
    """Try to evenly distribute layers across partitions.

    If the number of layers is not divisible by the number of partitions,
    the remaining layers are evenly distributed across all but the last
    partition. The last partition is excluded because it often contains an
    additional norm layer and we are attempting to balance compute.

    If `pp_size > 2` and the number of remaining layers is
    `0 < x <= pp_size - 2` then the remaining layers are evenly distributed
    across the middle partitions. The first and last partitions are excluded
    because they contain the input and output embeddings respectively and we
    are attempting to reduce maximum memory consumption across partitions.
    """
    partition_list_str = envs.VLLM_PP_LAYER_PARTITION
    if partition_list_str is not None:
        try:
            partitions = [int(layer) for layer in partition_list_str.split(",")]
        except ValueError as err:
            raise ValueError(
                "Invalid partition string: {}".format(partition_list_str)
            ) from err
        if len(partitions) != pp_size:
            raise ValueError(f"{len(partitions)=} does not match {pp_size=}.")
        if sum(partitions) != num_hidden_layers:
            raise ValueError(f"{sum(partitions)=} does not match {num_hidden_layers=}.")
    else:
        layers_per_partition = num_hidden_layers // pp_size
        partitions = [layers_per_partition for _ in range(pp_size)]

        if remaining_layers := num_hidden_layers % pp_size:
            for i in range(2, remaining_layers + 2):
                partitions[-i] += 1
            logger.info(
                "Hidden layers were unevenly partitioned: [%s]. "
                "This can be manually overridden using the "
                "VLLM_PP_LAYER_PARTITION environment variable",
                ",".join(str(p) for p in partitions),
            )

    start_layer = sum(partitions[:pp_rank])
    end_layer = start_layer + partitions[pp_rank]

    return (start_layer, end_layer)
```
**EN:** `get_pp_indices` retrieves state or computed results for this module. The docstring frames it as: Try to evenly distribute layers across partitions. It primarily works with arguments like `num_hidden_layers`, `pp_rank`, `pp_size`. Key calls include `sum`, `len`, `ValueError`.
**CN:** `get_pp_indices` 负责获取状态或计算结果。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `num_hidden_layers`, `pp_rank`, `pp_size` 这样的参数。 关键调用包括 `sum`, `len`, `ValueError`。

### Function `create_tcp_store` / 函数 `create_tcp_store`
```python
def create_tcp_store(
    host: str,
    port: int,
    listen_socket: socket.socket | None = None,
    **kwargs: Any,
) -> TCPStore:
    """Create a TCPStore, optionally taking ownership of ``listen_socket``."""
    if listen_socket is None:
        return TCPStore(host_name=host, port=port, **kwargs)

    listen_fd = listen_socket.detach()
    try:
        return TCPStore(
            host_name=host,
            port=port,
            master_listen_fd=listen_fd,
            **kwargs,
        )
    except Exception:
        socket.close(listen_fd)
        raise
```
**EN:** `create_tcp_store` constructs the required object or payload for this module. The docstring frames it as: Create a TCPStore, optionally taking ownership of ``listen_socket``. It primarily works with arguments like `host`, `port`, `listen_socket`, `**kwargs`. Key calls include `listen_socket.detach`, `TCPStore`, `socket.close`.
**CN:** `create_tcp_store` 负责构建所需对象或负载。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `host`, `port`, `listen_socket`, `**kwargs` 这样的参数。 关键调用包括 `listen_socket.detach`, `TCPStore`, `socket.close`。

### Class `StatelessProcessGroup` / 类 `StatelessProcessGroup`
```python
@dataclasses.dataclass
class StatelessProcessGroup:
    """A dataclass to hold a metadata store, and the rank, world_size of the
    group. Only use it to communicate metadata between processes.
    For data-plane communication, create NCCL-related objects.
    """

    rank: int
    world_size: int
    store: torch._C._distributed_c10d.Store

    data_expiration_seconds: int = 3600  # 1 hour

    # dst rank -> counter
    send_dst_counter: dict[int, int] = dataclasses.field(default_factory=dict)
    # src rank -> counter
    recv_src_counter: dict[int, int] = dataclasses.field(default_factory=dict)
    broadcast_send_counter: int = 0
    broadcast_recv_src_counter: dict[int, int] = dataclasses.field(default_factory=dict)

    # A deque to store the data entries, with key and timestamp.
    entries: deque[tuple[str, float]] = dataclasses.field(default_factory=deque)

    def __post_init__(self):
        assert self.rank < self.world_size
        self.send_dst_counter = {i: 0 for i in range(self.world_size)}
        self.recv_src_counter = {i: 0 for i in range(self.world_size)}
        self.broadcast_recv_src_counter = {i: 0 for i in range(self.world_size)}

    def send_obj(self, obj: Any, dst: int):
        """Send an object to a destination rank."""
        self.expire_data()
        key = f"send_to/{dst}/{self.send_dst_counter[dst]}"
        self.store.set(key, pickle.dumps(obj))
        self.send_dst_counter[dst] += 1
        self.entries.append((key, time.time()))

    def expire_data(self):
        """Expire data that is older than `data_expiration_seconds` seconds."""
        while self.entries:
            # check the oldest entry
            key, timestamp = self.entries[0]
            if time.time() - timestamp > self.data_expiration_seconds:
                self.store.delete_key(key)
                self.entries.popleft()
# ... truncated for analysis ...
            timeout=timedelta(seconds=store_timeout),
            use_libuv=False,  # for now: github.com/pytorch/pytorch/pull/150215
        )

        return StatelessProcessGroup(
            rank=rank,
            world_size=world_size,
            store=store,
            data_expiration_seconds=data_expiration_seconds,
        )
```
**EN:** Declares `StatelessProcessGroup`, a dataclass. It packages structured data fields such as `rank`, `world_size`, `store`, `data_expiration_seconds`, `send_dst_counter`. The docstring summarizes its role as: A dataclass to hold a metadata store, and the rank, world_size of the group. Only use it to communicate metadata between processes. For....
**CN:** 声明 `StatelessProcessGroup`，它是一个数据类。 它封装了 `rank`, `world_size`, `store`, `data_expiration_seconds`, `send_dst_counter` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Function `get_cached_tcp_store_client` / 函数 `get_cached_tcp_store_client`
```python
@functools.lru_cache(maxsize=1)
def get_cached_tcp_store_client(host: str, port: int) -> TCPStore:
    """Return a cached TCPStore client.

    Cached so that every call with the same ``(host, port)`` reuses the
    same connection.  A new ``(host, port)`` evicts the old entry.
    """
    return TCPStore(host, port, is_master=False, wait_for_workers=False)
```
**EN:** `get_cached_tcp_store_client` is a thin wrapper around `TCPStore`, exposing that operation through a module-level helper. The docstring frames it as: Return a cached TCPStore client. It primarily works with arguments like `host`, `port`. Key calls include `functools.lru_cache`, `TCPStore`.
**CN:** `get_cached_tcp_store_client` 是对 `TCPStore` 的轻量封装，通过模块级辅助函数暴露该操作。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `host`, `port` 这样的参数。 关键调用包括 `functools.lru_cache`, `TCPStore`。

### Function `init_gloo_process_group` / 函数 `init_gloo_process_group`
```python
def init_gloo_process_group(
    prefix_store: PrefixStore,
    group_rank: int,
    group_size: int,
    timeout: timedelta,
) -> ProcessGroup:
    """
    Stateless init ProcessGroup with gloo backend compatible with
    different torch versions.
    """
    with suppress_stdout():
        pg = ProcessGroup(
            prefix_store,
            group_rank,
            group_size,
        )
        from torch.distributed.distributed_c10d import ProcessGroupGloo

        backend_class = ProcessGroupGloo(
            prefix_store, group_rank, group_size, timeout=timeout
        )
        backend_type = ProcessGroup.BackendType.GLOO
        device = torch.device("cpu")
        pg._set_default_backend(backend_type)
        backend_class._set_sequence_number_for_group()

        pg._register_backend(device, backend_type, backend_class)
    return pg
```
**EN:** `init_gloo_process_group` initializes runtime state for this module. The docstring frames it as: Stateless init ProcessGroup with gloo backend compatible with different torch versions. It primarily works with arguments like `prefix_store`, `group_rank`, `group_size`, `timeout`. Key calls include `suppress_stdout`, `ProcessGroup`, `ProcessGroupGloo`.
**CN:** `init_gloo_process_group` 负责初始化运行时状态。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `prefix_store`, `group_rank`, `group_size`, `timeout` 这样的参数。 关键调用包括 `suppress_stdout`, `ProcessGroup`, `ProcessGroupGloo`。

### Function `stateless_init_torch_distributed_process_group` / 函数 `stateless_init_torch_distributed_process_group`
```python
def stateless_init_torch_distributed_process_group(
    host: str,
    port: int,
    rank: int,
    world_size: int,
    backend: str,
    group_name: str | None = None,
    return_store: bool = False,
    listen_socket: socket.socket | None = None,
) -> ProcessGroup | tuple[ProcessGroup, Store]:
    """
    A replacement for `torch.distributed.init_process_group` that does not
    pollute the global state. The created ProcessGroup object can be used for
    some operations such as `allreduce`, because it does not depend on the
    global rank. However, some operations such as `broadcast` cannot be used
    because it depends on the global rank.

    # TODO: ask for help from PyTorch team if we need the `broadcast` operation.

    This function is useful when we are not sure about the total number of
    processes in the process group. For example, we may have process
    1, 2, ..., 8 who want to communicate, and process 9 might be the same
    process as process 1, or it might be a different process; process 10
    might be the same process as process 5, or it might be a different process.
    In this case, how can we reliably form a communication channel within
    process 9 and 10, without affecting the communication channel within
    process 1, 2, ..., 8?

    One possible solution is to figure out if process 9 and 10 are the same
    as process 1 and 5 beforehand, and then form a communication channel
    based on the information, adjusting the ranks and world_size etc. However,
    figuring out the information is not always easy, and it will interfere
    with the main communication channel.

    Our solution is to always form a communication channel with process 1, 2,
    ..., 8, and then use this function to form another communication channel
    with process 9 and 10. This way, regardless of whether process 9 and 10
    are the same as process 1 and 5, the main communication channel is
    always formed with process 1, 2, ..., 8, and the additional communication
    channel is formed with process 9 and 10.

    When *listen_socket* is provided, the rendezvous step
    is skipped and a ``TCPStore`` server is created directly using the
    pre-bound socket.  This is useful for eliminating TOCTOU races
    between port allocation and binding.
# ... truncated for analysis ...
    if group_name is not None:
        from torch._C._distributed_c10d import _register_process_group

        pg._set_group_name(group_name)
        _register_process_group(group_name, pg)

    if return_store:
        return pg, store
    else:
        return pg
```
**EN:** `stateless_init_torch_distributed_process_group` implements a focused helper routine for this module. The docstring frames it as: A replacement for `torch.distributed.init_process_group` that does not pollute the global state. The created ProcessGroup object can be.... It primarily works with arguments like `host`, `port`, `rank`, `world_size`. Key calls include `get_tcp_uri`, `Backend`, `_get_default_timeout`.
**CN:** `stateless_init_torch_distributed_process_group` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `host`, `port`, `rank`, `world_size` 这样的参数。 关键调用包括 `get_tcp_uri`, `Backend`, `_get_default_timeout`。

### Function `stateless_destroy_torch_distributed_process_group` / 函数 `stateless_destroy_torch_distributed_process_group`
```python
def stateless_destroy_torch_distributed_process_group(pg: ProcessGroup) -> None:
    """
    Destroy ProcessGroup returned by
        stateless_init_torch_distributed_process_group().
    """
    pg.shutdown()
    _unregister_process_group(pg.group_name)
```
**EN:** `stateless_destroy_torch_distributed_process_group` implements a focused helper routine for this module. The docstring frames it as: Destroy ProcessGroup returned by stateless_init_torch_distributed_process_group(). It primarily works with arguments like `pg`. Key calls include `pg.shutdown`, `_unregister_process_group`.
**CN:** `stateless_destroy_torch_distributed_process_group` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `pg` 这样的参数。 关键调用包括 `pg.shutdown`, `_unregister_process_group`。

### Function `get_worker_rank_suffix` / 函数 `get_worker_rank_suffix`
```python
def get_worker_rank_suffix(global_rank: int | None = None) -> str:
    """Generate a descriptive rank suffix for worker identification.

    Returns a string like 'dp0_pp0_tp0_dcp0_ep0_rank0' including all
    parallel dimensions: DP, PP, TP, DCP, EP.

    Args:
        global_rank: Optional global rank to append. If not provided,
                     only parallel dimension ranks are included.

    Returns:
        A string suffix identifying the worker's position in the
        distributed topology.
    """
    from vllm.distributed.parallel_state import (
        get_dcp_group,
        get_dp_group,
        get_ep_group,
        get_pp_group,
        get_tp_group,
    )

    try:
        dp_rank = get_dp_group().rank_in_group
        pp_rank = get_pp_group().rank_in_group
        tp_rank = get_tp_group().rank_in_group
        dcp_rank = get_dcp_group().rank_in_group
        ep_rank = get_ep_group().rank_in_group

        suffix = f"dp{dp_rank}_pp{pp_rank}_tp{tp_rank}_dcp{dcp_rank}_ep{ep_rank}"
        if global_rank is not None:
            suffix = f"{suffix}_rank{global_rank}"
        return suffix
    except Exception:
        # Fallback if parallel state not initialized
        if global_rank is not None:
            return f"rank{global_rank}"
        return ""
```
**EN:** `get_worker_rank_suffix` retrieves state or computed results for this module. The docstring frames it as: Generate a descriptive rank suffix for worker identification. It primarily works with arguments like `global_rank`. Key calls include `get_dp_group`, `get_pp_group`, `get_tp_group`.
**CN:** `get_worker_rank_suffix` 负责获取状态或计算结果。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `global_rank` 这样的参数。 关键调用包括 `get_dp_group`, `get_pp_group`, `get_tp_group`。

## Key Concepts / 关键概念
- `sched_yield`: module-level helper or API entry / `sched_yield`：模块级辅助函数或 API 入口
- `ensure_divisibility`: module-level helper or API entry / `ensure_divisibility`：模块级辅助函数或 API 入口
- `divide`: module-level helper or API entry / `divide`：模块级辅助函数或 API 入口
- `split_tensor_along_last_dim`: module-level helper or API entry / `split_tensor_along_last_dim`：模块级辅助函数或 API 入口
- `get_pp_indices`: module-level helper or API entry / `get_pp_indices`：模块级辅助函数或 API 入口
- `create_tcp_store`: module-level helper or API entry / `create_tcp_store`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `functools`, `os`, `pickle`, `socket`, `sys`, `time`, `uuid`, `collections`, `collections.abc`, `datetime`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`, `torch.distributed.distributed_c10d`, `torch.distributed.rendezvous`, `torch._C._distributed_c10d`
- **Internal modules / 内部模块**: `vllm.envs`, `vllm.logger`, `vllm.utils.network_utils`, `vllm.utils.system_utils`, `vllm.distributed.parallel_state`, `vllm.platforms`
