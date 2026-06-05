# manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/cpu/manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CPUOffloadingManager` for the V1 `kv_offload/cpu` subsystem. / 为 V1 的 `kv_offload/cpu` 子系统实现 `CPUOffloadingManager`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from collections import OrderedDict
from collections.abc import Collection, Iterable
from typing import Literal

from vllm.v1.kv_offload.base import (
    LoadStoreSpec,
    OffloadingEvent,
    OffloadingManager,
    OffloadKey,
    PrepareStoreOutput,
    ReqContext,
)
from vllm.v1.kv_offload.cpu.common import CPULoadStoreSpec
from vllm.v1.kv_offload.cpu.policies.arc import ARCCachePolicy
from vllm.v1.kv_offload.cpu.policies.base import BlockStatus, CachePolicy
from vllm.v1.kv_offload.cpu.policies.lru import LRUCachePolicy

_CACHE_POLICIES: dict[str, type[CachePolicy]] = {
    "lru": LRUCachePolicy,
    "arc": ARCCachePolicy,
}
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_CACHE_POLICIES`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_CACHE_POLICIES`。

### `CPUOffloadingManager` class / `CPUOffloadingManager` 类
```python
class CPUOffloadingManager(OffloadingManager):
    """
    An OffloadingManager with a pluggable CachePolicy (LRU or ARC).

    The manager owns all shared logic: ref-counting, event emission,
    block pool management, and the prepare_store/complete_store skeletons.
    Policy-specific block organization and eviction decisions are delegated
    to the CachePolicy implementation.
    """
```
**EN:** Introduces the `CPUOffloadingManager` class on top of `OffloadingManager`. Core methods include `__init__`, `_get_num_free_blocks`, `_allocate_blocks`, `_free_block`, `_get_load_store_spec`, `lookup`. Docstring signal: An OffloadingManager with a pluggable CachePolicy (LRU or ARC).
**CN:** 这里定义 `CPUOffloadingManager` 类，其基类包括 `OffloadingManager`。核心方法包括 `__init__`, `_get_num_free_blocks`, `_allocate_blocks`, `_free_block`, `_get_load_store_spec`, `lookup`。

### `CPUOffloadingManager.__init__` method / `CPUOffloadingManager.__init__` 方法
```python
    def __init__(
        self,
        num_blocks: int,
        cache_policy: Literal["lru", "arc"] = "lru",
        enable_events: bool = False,
        store_threshold: int = 1,
        max_tracker_size: int = 64_000,
    ):
        self.medium: str = CPULoadStoreSpec.medium()
        self._num_blocks: int = num_blocks
        self._num_allocated_blocks: int = 0
        self._free_list: list[int] = []
        self.events: list[OffloadingEvent] | None = [] if enable_events else None
        policy_cls = _CACHE_POLICIES.get(cache_policy)
        if policy_cls is None:
            raise ValueError(
                f"Unknown cache policy: {cache_policy!r}. "
                f"Supported: {list(_CACHE_POLICIES)}"
            )
        self._policy: CachePolicy = policy_cls(cache_capacity=num_blocks)
        self.store_threshold: int = store_threshold
        self.max_tracker_size: int = max_tracker_size

        # Number of block references. It is ordered so can evict the LRU entry in O(1).
        self.counts: OrderedDict[OffloadKey, int] | None = (
            OrderedDict() if store_threshold >= 2 else None
        )
```
**EN:** This method initializes the object state within `CPUOffloadingManager`. Key calls include `medium`, `get`, `policy_cls`, `ValueError`, `OrderedDict`, `list`. It touches state such as `medium`, `_num_blocks`, `_num_allocated_blocks`, `_free_list`, `events`, `_policy`, `store_threshold`, `max_tracker_size`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`CPUOffloadingManager`。 关键调用包括 `medium`, `get`, `policy_cls`, `ValueError`, `OrderedDict`, `list`。 它会读写 `medium`, `_num_blocks`, `_num_allocated_blocks`, `_free_list`, `events`, `_policy`, `store_threshold`, `max_tracker_size` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CPUOffloadingManager._get_num_free_blocks` method / `CPUOffloadingManager._get_num_free_blocks` 方法
```python
    def _get_num_free_blocks(self) -> int:
        return len(self._free_list) + self._num_blocks - self._num_allocated_blocks
```
**EN:** This method implements `_get_num_free_blocks` within `CPUOffloadingManager`. Key calls include `len`.
**CN:** 该方法会实现 `_get_num_free_blocks`，其作用域位于`CPUOffloadingManager`。 关键调用包括 `len`。

### `CPUOffloadingManager._allocate_blocks` method / `CPUOffloadingManager._allocate_blocks` 方法
```python
    def _allocate_blocks(self, keys: list[OffloadKey]) -> list[BlockStatus]:
        num_fresh = min(len(keys), self._num_blocks - self._num_allocated_blocks)
        num_reused = len(keys) - num_fresh
        assert len(self._free_list) >= num_reused

        # allocate fresh blocks
        blocks: list[BlockStatus] = []
        for _ in range(num_fresh):
            blocks.append(BlockStatus(self._num_allocated_blocks))
            self._num_allocated_blocks += 1

        # allocate reused blocks
        for _ in range(num_reused):
            blocks.append(BlockStatus(self._free_list.pop()))
        return blocks
```
**EN:** This method implements `_allocate_blocks` within `CPUOffloadingManager`. Key calls include `min`, `range`, `len`, `append`, `BlockStatus`, `pop`. It touches state such as `_num_allocated_blocks`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_allocate_blocks`，其作用域位于`CPUOffloadingManager`。 关键调用包括 `min`, `range`, `len`, `append`, `BlockStatus`, `pop`。 它会读写 `_num_allocated_blocks` 等状态。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `CPUOffloadingManager._free_block` method / `CPUOffloadingManager._free_block` 方法
```python
    def _free_block(self, block: BlockStatus) -> None:
        self._free_list.append(block.block_id)
```
**EN:** This method implements `_free_block` within `CPUOffloadingManager`. Key calls include `append`.
**CN:** 该方法会实现 `_free_block`，其作用域位于`CPUOffloadingManager`。 关键调用包括 `append`。

### `CPUOffloadingManager._get_load_store_spec` method / `CPUOffloadingManager._get_load_store_spec` 方法
```python
    def _get_load_store_spec(
        self,
        keys: Iterable[OffloadKey],
        blocks: Iterable[BlockStatus],
    ) -> CPULoadStoreSpec:
        return CPULoadStoreSpec([block.block_id for block in blocks])
```
**EN:** This method implements `_get_load_store_spec` within `CPUOffloadingManager`. Key calls include `CPULoadStoreSpec`.
**CN:** 该方法会实现 `_get_load_store_spec`，其作用域位于`CPUOffloadingManager`。 关键调用包括 `CPULoadStoreSpec`。

### `CPUOffloadingManager.prepare_load` method / `CPUOffloadingManager.prepare_load` 方法
```python
    def prepare_load(
        self,
        keys: Collection[OffloadKey],
        req_context: ReqContext,
    ) -> LoadStoreSpec:
        blocks = []
        for key in keys:
            block = self._policy.get(key)
            assert block is not None, f"Block {key!r} not found in cache"
            assert block.is_ready, f"Block {key!r} is not ready for reading"
            block.ref_cnt += 1
            blocks.append(block)
        return self._get_load_store_spec(keys, blocks)
```
**EN:** This method prepares inputs and state within `CPUOffloadingManager`. Key calls include `_get_load_store_spec`, `get`, `append`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`CPUOffloadingManager`。 关键调用包括 `_get_load_store_spec`, `get`, `append`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `CPUOffloadingManager.complete_load` method / `CPUOffloadingManager.complete_load` 方法
```python
    def complete_load(
        self, keys: Collection[OffloadKey], req_context: ReqContext
    ) -> None:
        for key in keys:
            block = self._policy.get(key)
            assert block is not None, f"Block {key!r} not found"
            assert block.ref_cnt > 0, f"Block {key!r} ref_cnt is already 0"
            block.ref_cnt -= 1
```
**EN:** This method implements `complete_load` within `CPUOffloadingManager`. Key calls include `get`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `complete_load`，其作用域位于`CPUOffloadingManager`。 关键调用包括 `get`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `CPUOffloadingManager.prepare_store` method / `CPUOffloadingManager.prepare_store` 方法
```python
    def prepare_store(
        self,
        keys: Collection[OffloadKey],
        req_context: ReqContext,
    ) -> PrepareStoreOutput | None:
        if self.counts is not None:
            keys = [k for k in keys if self.counts.get(k, 0) >= self.store_threshold]
        # filter out blocks that are already stored
        keys_to_store = [k for k in keys if self._policy.get(k) is None]

        if not keys_to_store:
            return PrepareStoreOutput(
                keys_to_store=[],
                store_spec=self._get_load_store_spec([], []),
                evicted_keys=[],
            )

        num_blocks_to_evict = len(keys_to_store) - self._get_num_free_blocks()

        to_evict: list[OffloadKey] = []
        if num_blocks_to_evict > 0:
            # Blocks from the original input are excluded from eviction candidates:
            # a block that was already stored must remain in the cache after this call.
            protected = set(keys)
            evicted = self._policy.evict(num_blocks_to_evict, protected)
            if evicted is None:
                return None
            for key, block in evicted:
                self._free_block(block)
                to_evict.append(key)

        if to_evict and self.events is not None:
            self.events.append(
                OffloadingEvent(
                    keys=to_evict,
                    medium=self.medium,
                    removed=True,
                )
            )

        blocks = self._allocate_blocks(keys_to_store)
        assert len(blocks) == len(keys_to_store), (
            "Block pool did not allocate the expected number of blocks"
        )

        for key, block in zip(keys_to_store, blocks):
            self._policy.insert(key, block)

        # build store specs for allocated blocks
        store_spec = self._get_load_store_spec(keys_to_store, blocks)

        return PrepareStoreOutput(
            keys_to_store=keys_to_store,
            store_spec=store_spec,
            evicted_keys=to_evict,
        )
```
**EN:** This method prepares inputs and state within `CPUOffloadingManager`. Key calls include `_allocate_blocks`, `zip`, `_get_load_store_spec`, `PrepareStoreOutput`, `len`, `_get_num_free_blocks`. The control flow contains 5 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`CPUOffloadingManager`。 关键调用包括 `_allocate_blocks`, `zip`, `_get_load_store_spec`, `PrepareStoreOutput`, `len`, `_get_num_free_blocks`。 控制流包含 5 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `CPUOffloadingManager`: central class or interface in this module. / `CPUOffloadingManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `typing`
- Internal vLLM / 内部依赖: `vllm.v1.kv_offload.base`, `vllm.v1.kv_offload.cpu.common`, `vllm.v1.kv_offload.cpu.policies.arc`, `vllm.v1.kv_offload.cpu.policies.base`, `vllm.v1.kv_offload.cpu.policies.lru`
