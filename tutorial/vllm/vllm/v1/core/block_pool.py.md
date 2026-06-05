# block_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/block_pool.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `BlockHashToBlockMap`, `BlockPool` for the V1 `core` subsystem. / 为 V1 的 `core` 子系统实现 `BlockHashToBlockMap`, `BlockPool`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from collections.abc import Iterable, Sequence
from typing import Any

from vllm.distributed.kv_events import (
    MEDIUM_GPU,
    AllBlocksCleared,
    BlockRemoved,
    BlockStored,
    KVCacheEvent,
)
from vllm.logger import init_logger
from vllm.v1.core.kv_cache_metrics import KVCacheMetricsCollector
from vllm.v1.core.kv_cache_utils import (
    BlockHash,
    BlockHashList,
    BlockHashListWithBlockSize,
    BlockHashWithGroupId,
    ExternalBlockHash,
    FreeKVCacheBlockQueue,
    KVCacheBlock,
    generate_block_hash_extra_keys,
    get_block_hash,
    get_group_id,
    make_block_hash_with_group_id,
    maybe_convert_block_hash,
)
from vllm.v1.request import Request

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `BlockHashToBlockMap` class / `BlockHashToBlockMap` 类
```python
class BlockHashToBlockMap:
    """
    Cache of blocks that are used for prefix caching. It caches blocks
    from hash directly to a block or multiple blocks
    (i.e. {block_hash: KVCacheBlocks})
    - Mostly block_hash maps to a single KVCacheBlock, and KVCacheBlocks
        would simply be a KVCacheBlock.
    - Otherwise, KVCacheBlocks is a dict from {block_id: KVCacheBlock}

    A cached block is a full block with a block hash that can be used
    for prefix caching.
    The cached block may be used by running requests or in the
    free_block_queue that could potentially be evicted.

    NOTE #1: We currently don't de-duplicate the blocks in the cache,
    meaning that if a block becomes full and is cached, we don't check
    if there is already an identical block in the cache. This is because
    we want to make sure the allocated block IDs won't change so that
    block tables are append-only.
    NOTE #2: The union type is introduced in order to reduce GC costs
    from the inner dict.
    """
```
**EN:** Introduces the `BlockHashToBlockMap` class. Core methods include `__init__`, `get_one_block`, `insert`, `pop`, `__len__`, `_unexpected_blocks_type`. Docstring signal: Cache of blocks that are used for prefix caching.
**CN:** 这里定义 `BlockHashToBlockMap` 类。核心方法包括 `__init__`, `get_one_block`, `insert`, `pop`, `__len__`, `_unexpected_blocks_type`。

### `BlockHashToBlockMap.__init__` method / `BlockHashToBlockMap.__init__` 方法
```python
    def __init__(self):
        self._cache: dict[
            BlockHashWithGroupId, KVCacheBlock | dict[int, KVCacheBlock]
        ] = {}
```
**EN:** This method initializes the object state within `BlockHashToBlockMap`. It touches state such as `_cache`.
**CN:** 该方法会初始化对象状态，其作用域位于`BlockHashToBlockMap`。 它会读写 `_cache` 等状态。

### `BlockHashToBlockMap.insert` method / `BlockHashToBlockMap.insert` 方法
```python
    def insert(self, key: BlockHashWithGroupId, block: KVCacheBlock) -> None:
        """
        Inserts the KVCacheBlock to the cache
        """
        blocks = self._cache.get(key)
        if blocks is None:
            # When key is not found, attach a single block to the key
            self._cache[key] = block
        elif isinstance(blocks, KVCacheBlock):
            # If there's a block with the same key, merge the original block
            # and the new block into a dict
            self._cache[key] = {blocks.block_id: blocks, block.block_id: block}
        elif isinstance(blocks, dict):
            # If it's already a dict, simply insert the block
            blocks[block.block_id] = block
        else:
            self._unexpected_blocks_type(blocks)
```
**EN:** This method implements `insert` within `BlockHashToBlockMap`. The docstring frames it as: Inserts the KVCacheBlock to the cache Key calls include `get`, `isinstance`, `_unexpected_blocks_type`. It touches state such as `_cache`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `insert`，其作用域位于`BlockHashToBlockMap`。 关键调用包括 `get`, `isinstance`, `_unexpected_blocks_type`。 它会读写 `_cache` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BlockHashToBlockMap.pop` method / `BlockHashToBlockMap.pop` 方法
```python
    def pop(self, key: BlockHashWithGroupId, block_id: int) -> KVCacheBlock | None:
        """
        Checks if block_hash exists and pop block_id from the cache
        """
        blocks = self._cache.pop(key, None)
        if blocks is None:
            # block_hash not found in the cache
            return None
        # TODO(Jialin): If key is found, block_id should always present
        # in blocks. We currently keep the original behaviour for safety.
        #
        # Will add block_id == blocks.block_id assertion and
        # use del blocks[block_id] instead as followup.
        if isinstance(blocks, KVCacheBlock):
            if blocks.block_id == block_id:
                return blocks
            # If the single block ID doesn't match, we should put the
            # block back (it should happen rarely)
            self._cache[key] = blocks
            return None
        if isinstance(blocks, dict):
            # Try to pop block_id from the block dict, and if dict still
            # contain blocks, put back to the cache.
            block = blocks.pop(block_id, None)
            if len(blocks) > 0:
                self._cache[key] = blocks
            return block
        self._unexpected_blocks_type(blocks)
        return None
```
**EN:** This method implements `pop` within `BlockHashToBlockMap`. The docstring frames it as: Checks if block_hash exists and pop block_id from the cache Key calls include `pop`, `isinstance`, `_unexpected_blocks_type`, `len`. It touches state such as `_cache`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `pop`，其作用域位于`BlockHashToBlockMap`。 关键调用包括 `pop`, `isinstance`, `_unexpected_blocks_type`, `len`。 它会读写 `_cache` 等状态。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BlockPool` class / `BlockPool` 类
```python
class BlockPool:
    """BlockPool that manages KVCacheBlocks.
    It provides methods to allocate, free and cache the kv cache blocks. The
    free_block_queue stores the free blocks in eviction order to enable
    allocation, free, and cache eviction. The cached_block_hash_to_block
    maps between block hash and cached block to support finding cached blocks
    by their block hash.

    Args:
        num_gpu_blocks: The number of blocks in the pool.
        enable_caching: Whether to enable prefix caching.
        hash_block_size: The block size of which the block hashes are computed.
            The actual block size usually equals hash_block_size, but in cases
            where different KV cache groups have different block sizes, the
            actual block size can be a multiple of hash_block_size.
        enable_kv_cache_events: Whether to enable kv cache events.
        metrics_collector: Optional metrics collector for tracking block residency.
    """
```
**EN:** Introduces the `BlockPool` class. Core methods include `__init__`, `get_cached_block`, `cache_full_blocks`, `get_new_blocks`, `_maybe_evict_cached_block`, `touch`. Docstring signal: BlockPool that manages KVCacheBlocks.
**CN:** 这里定义 `BlockPool` 类。核心方法包括 `__init__`, `get_cached_block`, `cache_full_blocks`, `get_new_blocks`, `_maybe_evict_cached_block`, `touch`。

### `BlockPool.__init__` method / `BlockPool.__init__` 方法
```python
    def __init__(
        self,
        num_gpu_blocks: int,
        enable_caching: bool,
        hash_block_size: int,
        enable_kv_cache_events: bool = False,
        metrics_collector: KVCacheMetricsCollector | None = None,
    ):
        assert isinstance(num_gpu_blocks, int) and num_gpu_blocks > 0
        self.num_gpu_blocks = num_gpu_blocks
        self.enable_caching = enable_caching
        self.hash_block_size = hash_block_size
        # All kv-cache blocks.
        self.blocks: list[KVCacheBlock] = [
            KVCacheBlock(idx) for idx in range(num_gpu_blocks)
        ]
        # Free block queue that constructs and manipulates a doubly linked
        # list of free blocks (including eviction candidates when caching is
        # enabled).
        self.free_block_queue = FreeKVCacheBlockQueue(self.blocks)

        # Cache for block lookup
        self.cached_block_hash_to_block: BlockHashToBlockMap = BlockHashToBlockMap()

        # To represent a placeholder block with block_id=0.
        # The ref_cnt of null_block is not maintained, needs special care to
        # avoid freeing it.
        self.null_block = self.free_block_queue.popleft()
        self.null_block.is_null = True

        self.enable_kv_cache_events = enable_kv_cache_events
        self.kv_event_queue: list[KVCacheEvent] = []

        self.metrics_collector = metrics_collector
```
**EN:** This method initializes the object state within `BlockPool`. Key calls include `FreeKVCacheBlockQueue`, `BlockHashToBlockMap`, `popleft`, `isinstance`, `KVCacheBlock`, `range`. It touches state such as `num_gpu_blocks`, `enable_caching`, `hash_block_size`, `blocks`, `free_block_queue`, `cached_block_hash_to_block`, `null_block`, `enable_kv_cache_events`.
**CN:** 该方法会初始化对象状态，其作用域位于`BlockPool`。 关键调用包括 `FreeKVCacheBlockQueue`, `BlockHashToBlockMap`, `popleft`, `isinstance`, `KVCacheBlock`, `range`。 它会读写 `num_gpu_blocks`, `enable_caching`, `hash_block_size`, `blocks`, `free_block_queue`, `cached_block_hash_to_block`, `null_block`, `enable_kv_cache_events` 等状态。

### `BlockPool.cache_full_blocks` method / `BlockPool.cache_full_blocks` 方法
```python
    def cache_full_blocks(
        self,
        request: Request,
        blocks: list[KVCacheBlock],
        num_cached_blocks: int,
        num_full_blocks: int,
        block_size: int,
        kv_cache_group_id: int,
        block_mask: list[bool] | None = None,
    ) -> None:
        """Cache a list of full blocks for prefix caching.
        This function takes a list of blocks that will have their block hash
        metadata to be updated and cached. Given a request, it updates the
        metadata for each block and caching it in the
        `cached_block_hash_to_block`.
        The block hashes values are computed by the Request object immediately
        when it is created and when new tokens are appended.

        Args:
            request: The request to cache the blocks.
            blocks: All blocks in the request.
            num_cached_blocks: The number of blocks that are already cached.
            num_full_blocks: The number of blocks that are full and should
                be cached after this function.
            block_size: Number of tokens in each block.
            kv_cache_group_id: The id of the KV cache group.
            block_mask: Optional mask aligned with
                ``blocks[num_cached_blocks:num_full_blocks]``. When provided,
                blocks where the mask is False are skipped (treated like null
                blocks). Used by groups whose ``find_longest_cache_hit`` only
                consults a subset of blocks (e.g. SWA tail-window), so blocks
                that can never serve a hit stay out of the prefix-cache hash
                map.
        """
        if num_cached_blocks >= num_full_blocks:
            return
        new_full_blocks = blocks[num_cached_blocks:num_full_blocks]
        assert len(request.block_hashes) >= num_full_blocks
        assert block_mask is None or len(block_mask) == len(new_full_blocks)
        if block_size == self.hash_block_size:
            # Common case.
            block_hashes: BlockHashList = request.block_hashes
        else:
            # block_size is a multiple of hash_block_size. This happens when
            # different KV cache groups have different block sizes.
            assert block_size % self.hash_block_size == 0
            # Recalculate block_hashes at the granularity of block_size, using
            # the original block_hashes (at the granularity of hash_block_size).
            block_hashes = BlockHashListWithBlockSize(
                request.block_hashes, self.hash_block_size, block_size
    # ... omitted for brevity ...
                    continue
                block_start = i * block_size
                block_end = block_start + block_size
                extra_keys, curr_mm_idx = generate_block_hash_extra_keys(
                    request, block_start, block_end, curr_mm_idx
                )
                extra_keys_list.append(extra_keys)

            self.kv_event_queue.append(
                BlockStored(
                    block_hashes=new_hashes,
                    parent_block_hash=parent_block_hash,
                    token_ids=request.all_token_ids[start_token_idx:end_token_idx],
                    block_size=block_size,
                    lora_id=request.lora_request.adapter_id
                    if request.lora_request
                    else None,
                    medium=MEDIUM_GPU,
                    lora_name=request.lora_request.name
                    if request.lora_request
                    else None,
                    extra_keys=extra_keys_list if extra_keys_list else None,
                    group_idx=kv_cache_group_id,
                )
            )
```
**EN:** This method implements `cache_full_blocks` within `BlockPool`. The docstring frames it as: Cache a list of full blocks for prefix caching. Key calls include `enumerate`, `len`, `BlockHashListWithBlockSize`, `make_block_hash_with_group_id`, `insert`, `range`. The control flow contains 12 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `cache_full_blocks`，其作用域位于`BlockPool`。 关键调用包括 `enumerate`, `len`, `BlockHashListWithBlockSize`, `make_block_hash_with_group_id`, `insert`, `range`。 控制流包含 12 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `BlockPool.free_blocks` method / `BlockPool.free_blocks` 方法
```python
    def free_blocks(self, ordered_blocks: Iterable[KVCacheBlock]) -> None:
        """Free a list of blocks. The blocks should be ordered by their
        eviction priority, where the first block will be evicted first.

        Args:
            ordered_blocks: A list of blocks to free ordered by their eviction
                priority.
        """
        # Materialize the iterable to allow multiple passes.
        blocks_list = list(ordered_blocks)
        for block in blocks_list:
            block.ref_cnt -= 1
        self.free_block_queue.append_n(
            [block for block in blocks_list if block.ref_cnt == 0 and not block.is_null]
        )
```
**EN:** This method implements `free_blocks` within `BlockPool`. The docstring frames it as: Free a list of blocks. Key calls include `list`, `append_n`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `free_blocks`，其作用域位于`BlockPool`。 关键调用包括 `list`, `append_n`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `BlockHashToBlockMap`: central class or interface in this module. / `BlockHashToBlockMap`：本模块中的核心类或接口。
- `BlockPool`: central class or interface in this module. / `BlockPool`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `typing`
- Internal vLLM / 内部依赖: `vllm.distributed.kv_events`, `vllm.logger`, `vllm.v1.core.kv_cache_metrics`, `vllm.v1.core.kv_cache_utils`, `vllm.v1.request`
