# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `StatelessProcessGroup`, `ensure_divisibility`, and `divide`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `StatelessProcessGroup`、`ensure_divisibility` 和 `divide` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 11-21: module setup and imports / 模块初始化与导入
```python
import dataclasses
import pickle
import time
from collections import deque
from collections.abc import Sequence
from typing import Any

import torch
from torch.distributed import TCPStore

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `dataclasses`, `pickle`, `time`, `collections`, `collections.abc`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`pickle`、`time`、`collections`、`collections.abc` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 23-23: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 26-30: `ensure_divisibility` implementation / `ensure_divisibility` 实现
```python
def ensure_divisibility(numerator, denominator) -> None:
    """Ensure that numerator is divisible by the denominator."""
    assert numerator % denominator == 0, "{} is not divisible by {}".format(
        numerator, denominator
    )
```
**EN:** This block defines function `ensure_divisibility`. Ensure that numerator is divisible by the denominator. Key calls include `format`. Parameters such as `numerator`, and `denominator` drive the behavior in this section.
**CN:** 该代码块定义了函数 `ensure_divisibility`。 它用于处理 ensure divisibility 相关逻辑。 关键调用包括 `format`。 本段逻辑主要由 `numerator` 和 `denominator` 等参数驱动。

### Lines 33-37: `divide` implementation / `divide` 实现
```python
def divide(numerator: int, denominator: int) -> int:
    """Ensure that numerator is divisible by the denominator and return
    the division value."""
    ensure_divisibility(numerator, denominator)
    return numerator // denominator
```
**EN:** This block defines function `divide`. Ensure that numerator is divisible by the denominator and return the division value. Key calls include `ensure_divisibility`. Parameters such as `numerator`, and `denominator` drive the behavior in this section.
**CN:** 该代码块定义了函数 `divide`。 它用于处理 divide 相关逻辑。 关键调用包括 `ensure_divisibility`。 本段逻辑主要由 `numerator` 和 `denominator` 等参数驱动。

### Lines 40-65: `split_tensor_along_last_dim` implementation / `split_tensor_along_last_dim` 实现
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

    return tuple(tensor_list)
```
**EN:** This block defines function `split_tensor_along_last_dim`. Split a tensor along its last dimension. Arguments: tensor: input tensor. Key calls include `divide`, `torch.split`, `tuple`, `tensor.dim`, and `tensor.size`. The implementation branches on conditions. Parameters such as `tensor`, `num_partitions`, and `contiguous_split_chunks` drive the behavior in this section.
**CN:** 该代码块定义了函数 `split_tensor_along_last_dim`。 它用于拆分tensor along last dim。 关键调用包括 `divide`、`torch.split`、`tuple`、`tensor.dim` 和 `tensor.size`。 实现中包含条件分支。 本段逻辑主要由 `tensor`、`num_partitions` 和 `contiguous_split_chunks` 等参数驱动。

### Lines 69-74: `StatelessProcessGroup` class overview / `StatelessProcessGroup` 类概览
```python
class StatelessProcessGroup:
    """A dataclass to hold a metadata store, and the rank, world_size of the
    group. Only use it to communicate metadata between processes.
    For data-plane communication, create NCCL-related objects.
    """
```
**EN:** This block defines class `StatelessProcessGroup`. A dataclass to hold a metadata store, and the rank, world_size of the group. Only use it to communicate metadata between processes.
**CN:** 该代码块定义了类 `StatelessProcessGroup`。 它用于封装 stateless process group 相关行为。

### Lines 75-88: supporting statements / 辅助语句
```python
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
```
**EN:** This block gathers supporting statements inside `StatelessProcessGroup`. It updates names such as `rank`, `world_size`, `store`, `data_expiration_seconds`, `send_dst_counter`, and `recv_src_counter`. The code collaborates with `dataclasses.field`.
**CN:** 该代码块汇集了位于 `StatelessProcessGroup` 内部的辅助语句。 它会更新 `rank`、`world_size`、`store`、`data_expiration_seconds`、`send_dst_counter` 和 `recv_src_counter` 等名称。 代码会与 `dataclasses.field` 协同工作。

### Lines 90-94: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        assert self.rank < self.world_size
        self.send_dst_counter = {i: 0 for i in range(self.world_size)}
        self.recv_src_counter = {i: 0 for i in range(self.world_size)}
        self.broadcast_recv_src_counter = {i: 0 for i in range(self.world_size)}
```
**EN:** This block defines method `__post_init__` on `StatelessProcessGroup`. It post-processes init. Key calls include `range`.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `range`。

### Lines 96-102: `send_obj` implementation / `send_obj` 实现
```python
    def send_obj(self, obj: Any, dst: int):
        """Send an object to a destination rank."""
        self.expire_data()
        key = f"send_to/{dst}/{self.send_dst_counter[dst]}"
        self.store.set(key, pickle.dumps(obj))
        self.send_dst_counter[dst] += 1
        self.entries.append((key, time.perf_counter()))
```
**EN:** This block defines method `send_obj` on `StatelessProcessGroup`. Send an object to a destination rank. Key calls include `self.expire_data`, `self.store.set`, `self.entries.append`, `pickle.dumps`, and `time.perf_counter`. Parameters such as `obj`, and `dst` drive the behavior in this section.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `send_obj`。 它用于处理 send obj 相关逻辑。 关键调用包括 `self.expire_data`、`self.store.set`、`self.entries.append`、`pickle.dumps` 和 `time.perf_counter`。 本段逻辑主要由 `obj` 和 `dst` 等参数驱动。

### Lines 104-113: `expire_data` implementation / `expire_data` 实现
```python
    def expire_data(self) -> None:
        """Expire data that is older than `data_expiration_seconds` seconds."""
        while self.entries:
            # check the oldest entry
            key, timestamp = self.entries[0]
            if time.perf_counter() - timestamp > self.data_expiration_seconds:
                self.store.delete_key(key)
                self.entries.popleft()
            else:
                break
```
**EN:** This block defines method `expire_data` on `StatelessProcessGroup`. Expire data that is older than `data_expiration_seconds` seconds. Key calls include `self.store.delete_key`, `self.entries.popleft`, and `time.perf_counter`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `expire_data`。 它用于处理 expire data 相关逻辑。 关键调用包括 `self.store.delete_key`、`self.entries.popleft` 和 `time.perf_counter`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 115-121: `recv_obj` implementation / `recv_obj` 实现
```python
    def recv_obj(self, src: int) -> Any:
        """Receive an object from a source rank."""
        obj = pickle.loads(
            self.store.get(f"send_to/{self.rank}/{self.recv_src_counter[src]}")
        )
        self.recv_src_counter[src] += 1
        return obj
```
**EN:** This block defines method `recv_obj` on `StatelessProcessGroup`. Receive an object from a source rank. Key calls include `pickle.loads`, and `self.store.get`. Parameters such as `src` drive the behavior in this section.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `recv_obj`。 它用于处理 recv obj 相关逻辑。 关键调用包括 `pickle.loads` 和 `self.store.get`。 本段逻辑主要由 `src` 等参数驱动。

### Lines 123-139: `broadcast_obj` implementation / `broadcast_obj` 实现
```python
    def broadcast_obj(self, obj: Any | None, src: int) -> Any:
        """Broadcast an object from a source rank to all other ranks.
        It does not clean up after all ranks have received the object.
        Use it for limited times, e.g., for initialization.
        """
        if self.rank == src:
            self.expire_data()
            key = f"broadcast_from/{src}/" f"{self.broadcast_send_counter}"
            self.store.set(key, pickle.dumps(obj))
            self.broadcast_send_counter += 1
            self.entries.append((key, time.perf_counter()))
            return obj
        else:
            key = f"broadcast_from/{src}/" f"{self.broadcast_recv_src_counter[src]}"
            recv_obj = pickle.loads(self.store.get(key))
            self.broadcast_recv_src_counter[src] += 1
            return recv_obj
```
**EN:** This block defines method `broadcast_obj` on `StatelessProcessGroup`. Broadcast an object from a source rank to all other ranks. It does not clean up after all ranks have received the object. Key calls include `self.expire_data`, `self.store.set`, `self.entries.append`, `pickle.loads`, and `pickle.dumps`. The implementation branches on conditions. Parameters such as `obj`, and `src` drive the behavior in this section.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `broadcast_obj`。 它用于处理 broadcast obj 相关逻辑。 关键调用包括 `self.expire_data`、`self.store.set`、`self.entries.append`、`pickle.loads` 和 `pickle.dumps`。 实现中包含条件分支。 本段逻辑主要由 `obj` 和 `src` 等参数驱动。

### Lines 141-151: `all_gather_obj` implementation / `all_gather_obj` 实现
```python
    def all_gather_obj(self, obj: Any) -> list[Any]:
        """All gather an object from all ranks."""
        gathered_objs = []
        for i in range(self.world_size):
            if i == self.rank:
                gathered_objs.append(obj)
                self.broadcast_obj(obj, src=self.rank)
            else:
                recv_obj = self.broadcast_obj(None, src=i)
                gathered_objs.append(recv_obj)
        return gathered_objs
```
**EN:** This block defines method `all_gather_obj` on `StatelessProcessGroup`. All gather an object from all ranks. Key calls include `range`, `gathered_objs.append`, and `self.broadcast_obj`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `obj` drive the behavior in this section.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `all_gather_obj`。 它用于处理 all gather obj 相关逻辑。 关键调用包括 `range`、`gathered_objs.append` 和 `self.broadcast_obj`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `obj` 等参数驱动。

### Lines 153-159: `barrier` implementation / `barrier` 实现
```python
    def barrier(self):
        """A barrier to synchronize all ranks."""
        for i in range(self.world_size):
            if i == self.rank:
                self.broadcast_obj(None, src=self.rank)
            else:
                self.broadcast_obj(None, src=i)
```
**EN:** This block defines method `barrier` on `StatelessProcessGroup`. A barrier to synchronize all ranks. Key calls include `range`, and `self.broadcast_obj`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `barrier`。 它用于处理 barrier 相关逻辑。 关键调用包括 `range` 和 `self.broadcast_obj`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 161-196: `create` implementation / `create` 实现
```python
    @staticmethod
    def create(
        host: str,
        port: int,
        rank: int,
        world_size: int,
        data_expiration_seconds: int = 3600,
    ) -> "StatelessProcessGroup":
        """A replacement for `torch.distributed.init_process_group` that does not
        pollute the global state.

        If we have process A and process B called `torch.distributed.init_process_group`
        to form a group, and then we want to form another group with process A, B, C,
        D, it is not possible in PyTorch, because process A and process B have already
        formed a group, and process C and process D cannot join that group. This
        function is a workaround for this issue.

        `torch.distributed.init_process_group` is a global call, while this function
        is a stateless call. It will return a `StatelessProcessGroup` object that can be
        used for exchanging metadata. With this function, process A and process B
        can call `StatelessProcessGroup.create` to form a group, and then process A, B,
        C, and D can call `StatelessProcessGroup.create` to form another group.
        """  # noqa
        store = TCPStore(
            host_name=host,
            port=port,
            world_size=world_size,
            is_master=(rank == 0),
        )

        return StatelessProcessGroup(
            rank=rank,
            world_size=world_size,
            store=store,
            data_expiration_seconds=data_expiration_seconds,
        )
```
**EN:** This block defines method `create` on `StatelessProcessGroup`. A replacement for `torch.distributed.init_process_group` that does not pollute the global state. If we have process A and process B called `torch.distributed.init_process_group` to form a group, and then we want to form another group with process A, B, C, D, it is not possible in PyTorch, because process A and process B have already formed a group, and process C and process D cannot join that group. Key calls include `TCPStore`, and `StatelessProcessGroup`. Parameters such as `host`, `port`, `rank`, `world_size`, and `data_expiration_seconds` drive the behavior in this section.
**CN:** 该代码块定义了 `StatelessProcessGroup` 的方法 `create`。 它用于创建函数。 关键调用包括 `TCPStore` 和 `StatelessProcessGroup`。 本段逻辑主要由 `host`、`port`、`rank`、`world_size` 和 `data_expiration_seconds` 等参数驱动。

## Key Concepts / 关键概念
- `ensure_divisibility`: Ensure that numerator is divisible by the denominator. / 顶层函数，用于处理 ensure divisibility 相关逻辑。
- `divide`: Ensure that numerator is divisible by the denominator and return the division value. / 顶层函数，用于处理 divide 相关逻辑。
- `split_tensor_along_last_dim`: Split a tensor along its last dimension. / 顶层函数，用于拆分tensor along last dim。
- `StatelessProcessGroup`: A dataclass to hold a metadata store, and the rank, world_size of the group. / 核心类，用于封装 stateless process group 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `pickle`, `time`, `collections`, `collections.abc`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 196
