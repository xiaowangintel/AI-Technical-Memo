# kv_cache_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/kv_cache_manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `KVCacheBlocks`, `KVCacheManager` for the V1 `core` subsystem. / 为 V1 的 `core` 子系统实现 `KVCacheBlocks`, `KVCacheManager`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import itertools
from collections.abc import Sequence
from dataclasses import dataclass
from typing import Literal, overload

from vllm.distributed.kv_events import BlockStored, KVCacheEvent
from vllm.logger import init_logger
from vllm.v1.core.kv_cache_coordinator import get_kv_cache_coordinator
from vllm.v1.core.kv_cache_metrics import KVCacheMetricsCollector
from vllm.v1.core.kv_cache_utils import KVCacheBlock
from vllm.v1.kv_cache_interface import (
    KVCacheConfig,
    get_kv_cache_spec_kind,
    get_kv_cache_spec_sliding_window,
)
from vllm.v1.metrics.stats import PrefixCacheStats
from vllm.v1.request import Request

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `KVCacheBlocks` class / `KVCacheBlocks` 类
```python
@dataclass
class KVCacheBlocks:
    """
    The allocation result of KVCacheManager, work as the interface between
    Scheduler and KVCacheManager, to hide KVCacheManager's internal data
    structure from the Scheduler.
    """

    blocks: tuple[Sequence[KVCacheBlock], ...]
    """
    `blocks[i][j]` refers to the i-th kv_cache_group
    and the j-th block of tokens.We don't use block of
    tokens as the outer dimension because it assumes all
    kv_cache_groups have the same number of blocks, which is true for now but
    will be broken if we want to give different block_size to different
    kv_cache_groups in the future.

    Each single type KVCacheBlocks could be represented as:
    - list[KVCacheBlock] for more than one KVCacheBlock
    - an empty tuple for requests without KVCacheBlock
      (a precomputed KVCacheBlocks is in KVCacheManager to avoid GC overhead)
    """
```
**EN:** Uses `@dataclass` to package related state for `KVCacheBlocks`. Typical fields include `blocks`.
**CN:** `KVCacheBlocks` 使用 `@dataclass` 打包相关状态。典型字段包括 `blocks`。

### `KVCacheBlocks.get_block_ids` method / `KVCacheBlocks.get_block_ids` 方法
```python
    def get_block_ids(
        self,
        allow_none: bool = False,
    ) -> tuple[list[int], ...] | None:
        """
        Converts the KVCacheBlocks instance to block_ids.

        Returns:
            tuple[list[int], ...]: A tuple of lists where:
                - the outer tuple corresponds to KV cache groups
                - each inner list contains the block_ids of the blocks in that
                  group
        """
        if allow_none and all(len(group) == 0 for group in self.blocks):
            return None
        return tuple([blk.block_id for blk in group] for group in self.blocks)
```
**EN:** This method returns or derives a value within `KVCacheBlocks`. The docstring frames it as: Converts the KVCacheBlocks instance to block_ids. Key calls include `tuple`, `all`, `len`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`KVCacheBlocks`。 关键调用包括 `tuple`, `all`, `len`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheBlocks.get_unhashed_block_ids_all_groups` method / `KVCacheBlocks.get_unhashed_block_ids_all_groups` 方法
```python
    def get_unhashed_block_ids_all_groups(self) -> list[list[int]]:
        """Get block_ids of unhashed blocks from KVCacheBlocks instance."""
        # Skip padding blocks.
        return [
            [
                block.block_id
                for block in group
                if block.block_hash is None and not block.is_null
            ]
            for group in self.blocks
        ]
```
**EN:** This method returns or derives a value within `KVCacheBlocks`. The docstring frames it as: Get block_ids of unhashed blocks from KVCacheBlocks instance.
**CN:** 该方法会返回或推导一个值，其作用域位于`KVCacheBlocks`。

### `KVCacheBlocks.new_empty` method / `KVCacheBlocks.new_empty` 方法
```python
    def new_empty(self) -> "KVCacheBlocks":
        """
        Creates a new KVCacheBlocks instance with no blocks.
        """
        return KVCacheBlocks(tuple(() for _ in range(len(self.blocks))))
```
**EN:** This method implements `new_empty` within `KVCacheBlocks`. The docstring frames it as: Creates a new KVCacheBlocks instance with no blocks. Key calls include `KVCacheBlocks`, `tuple`, `range`, `len`.
**CN:** 该方法会实现 `new_empty`，其作用域位于`KVCacheBlocks`。 关键调用包括 `KVCacheBlocks`, `tuple`, `range`, `len`。

### `KVCacheManager` class / `KVCacheManager` 类
```python
class KVCacheManager:
```
**EN:** Introduces the `KVCacheManager` class. Core methods include `__init__`, `usage`, `make_prefix_cache_stats`, `get_computed_blocks`, `allocate_slots`, `free`.
**CN:** 这里定义 `KVCacheManager` 类。核心方法包括 `__init__`, `usage`, `make_prefix_cache_stats`, `get_computed_blocks`, `allocate_slots`, `free`。

### `KVCacheManager.__init__` method / `KVCacheManager.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_config: KVCacheConfig,
        max_model_len: int,
        hash_block_size: int,
        max_num_batched_tokens: int | None = None,
        enable_caching: bool = True,
        use_eagle: bool = False,
        log_stats: bool = False,
        enable_kv_cache_events: bool = False,
        dcp_world_size: int = 1,
        pcp_world_size: int = 1,
        metrics_collector: KVCacheMetricsCollector | None = None,
    ) -> None:
        self.max_model_len = max_model_len
        # When unset, fall back to `max_model_len` so the recycling-aware cap
        # collapses to the prior (uncapped) admission behavior. The scheduler
        # always supplies the real value at runtime.
        if max_num_batched_tokens is None:
            max_num_batched_tokens = max_model_len

        self.enable_caching = enable_caching
        self.use_eagle = use_eagle
        self.log_stats = log_stats
        self.metrics_collector = metrics_collector
        # FIXME: make prefix cache stats conditional on log_stats. We still need
        # this comment because when the log stats is enabled there are still
        # potential configs we could expose in the future.
        self.prefix_cache_stats = PrefixCacheStats() if log_stats else None

        self.coordinator = get_kv_cache_coordinator(
            kv_cache_config=kv_cache_config,
            max_model_len=self.max_model_len,
            max_num_batched_tokens=max_num_batched_tokens,
            use_eagle=self.use_eagle,
            enable_caching=self.enable_caching,
            enable_kv_cache_events=enable_kv_cache_events,
            dcp_world_size=dcp_world_size,
            pcp_world_size=pcp_world_size,
            hash_block_size=hash_block_size,
            metrics_collector=self.metrics_collector,
        )
        self.num_kv_cache_groups = len(kv_cache_config.kv_cache_groups)
        self.block_pool = self.coordinator.block_pool
        self.kv_cache_config = kv_cache_config
        self.kv_cache_event_metadata = tuple(
            (
                get_kv_cache_spec_kind(group.kv_cache_spec).value,
                get_kv_cache_spec_sliding_window(group.kv_cache_spec),
            )
            for group in kv_cache_config.kv_cache_groups
        )

        # Pre-constructed KVCacheBlocks with no blocks, callers should use this
        # via create_kv_cache_blocks instead of creating new ones to avoid GC
        # overhead.
        #
        # We use nested tuples to ensure the empty KVCacheBlocks is immutable.
        self.empty_kv_cache_blocks = KVCacheBlocks(
            tuple(() for _ in range(self.num_kv_cache_groups))
        )
```
**EN:** This method initializes the object state within `KVCacheManager`. Key calls include `get_kv_cache_coordinator`, `len`, `tuple`, `KVCacheBlocks`, `PrefixCacheStats`, `get_kv_cache_spec_sliding_window`. It touches state such as `max_model_len`, `enable_caching`, `use_eagle`, `log_stats`, `metrics_collector`, `prefix_cache_stats`, `coordinator`, `num_kv_cache_groups`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`KVCacheManager`。 关键调用包括 `get_kv_cache_coordinator`, `len`, `tuple`, `KVCacheBlocks`, `PrefixCacheStats`, `get_kv_cache_spec_sliding_window`。 它会读写 `max_model_len`, `enable_caching`, `use_eagle`, `log_stats`, `metrics_collector`, `prefix_cache_stats`, `coordinator`, `num_kv_cache_groups` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheManager.get_computed_blocks` method / `KVCacheManager.get_computed_blocks` 方法
```python
    def get_computed_blocks(self, request: Request) -> tuple[KVCacheBlocks, int]:
        """Get the computed (cached) blocks for the request.
        Note that the computed blocks must be full.

        Args:
            request: The request to get the computed blocks.

        Returns:
            A tuple containing:
                - A list of blocks that are computed for the request.
                - The number of computed tokens.
        """
        # We skip finding the prefix cache hit when prefix caching is
        # disabled or the request is marked as skipping kv cache read
        # (which happens when the request requires prompt logprobs
        # or calls a pooling model with all pooling).
        if not self.enable_caching or request.skip_reading_prefix_cache:
            return self.empty_kv_cache_blocks, 0

        # NOTE: When all tokens hit the cache, we must recompute the last token
        # to obtain logits. Thus, set max_cache_hit_length to prompt_length - 1.
        # This can trigger recomputation of an entire block, rather than just
        # the single last token, because allocate_slots() requires
        # num_computed_tokens to be block-size aligned. Removing this limitation
        # could slightly improve performance in the future.
        max_cache_hit_length = request.num_tokens - 1
        computed_blocks, num_new_computed_tokens = (
            self.coordinator.find_longest_cache_hit(
                request.block_hashes, max_cache_hit_length
            )
        )

        if self.log_stats:
            assert self.prefix_cache_stats is not None
            self.prefix_cache_stats.record(
                num_tokens=request.num_tokens,
                num_hits=num_new_computed_tokens,
                preempted=request.num_preemptions > 0,
            )

        return self.create_kv_cache_blocks(computed_blocks), num_new_computed_tokens
```
**EN:** This method returns or derives a value within `KVCacheManager`. The docstring frames it as: Get the computed (cached) blocks for the request. Key calls include `find_longest_cache_hit`, `record`, `create_kv_cache_blocks`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`KVCacheManager`。 关键调用包括 `find_longest_cache_hit`, `record`, `create_kv_cache_blocks`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheManager.allocate_slots` method / `KVCacheManager.allocate_slots` 方法
```python
    def allocate_slots(
        self,
        request: Request,
        num_new_tokens: int,
        num_new_computed_tokens: int = 0,
        new_computed_blocks: KVCacheBlocks | None = None,
        num_lookahead_tokens: int = 0,
        num_external_computed_tokens: int = 0,
        delay_cache_blocks: bool = False,
        num_encoder_tokens: int = 0,
        full_sequence_must_fit: bool = False,
    ) -> KVCacheBlocks | None:
        """Add slots for a request with new tokens to append.

        Args:
            request: The request to allocate slots.
            num_new_tokens: The number of new tokens to be allocated and computed.
            num_new_computed_tokens: The number of new computed tokens just
                hitting the prefix caching, excluding external tokens.
            new_computed_blocks: The cached blocks for the above new computed
                tokens, grouped as a tuple by kv cache groups.
            num_lookahead_tokens: The number of speculative tokens to allocate.
                This is used by spec decode proposers with kv-cache such
                as eagle.
            num_external_computed_tokens: The number of tokens that their
                KV caches are not cached by vLLM but cached by the connector.
            delay_cache_blocks: Whether to skip caching the blocks. This is
                used by P/D when allocating blocks used in a KV transfer
                which will complete in a future step.
            num_encoder_tokens: The number of encoder tokens to allocate for
                cross-attention in encoder-decoder models(e.g., Whisper).
                For decoder-only models, this should be 0.
            full_sequence_must_fit: Only allocate blocks if the KV cache has enough
                free blocks to hold the full sequence, accounting for prefix cache hits
                and sliding window. Used as an admission gate to prevent over-admitting
                requests when chunked prefill would otherwise only check the first chunk

        Blocks layout:
        ```
        ----------------------------------------------------------------------
        | < comp > | < new_comp > | < ext_comp >  | < new >  | < lookahead > |
        ----------------------------------------------------------------------
                                                  |   < to be computed >     |
        ----------------------------------------------------------------------
                                  |            < to be allocated >           |
        ----------------------------------------------------------------------
                                  | < to be cached (roughly, |
                                  | details below)>          |
        ----------------------------------------------------------------------
        | Prefix-cached tokens from either vLLM   |
    # ... omitted for brevity ...

        new_blocks = self.coordinator.allocate_new_blocks(
            request.request_id,
            num_tokens_need_slot,
            num_tokens_main_model,
            num_encoder_tokens,
        )

        # P/D: delay caching blocks if we have to recv from
        # remote. Update state for locally cached blocks.
        if not self.enable_caching or delay_cache_blocks:
            return self.create_kv_cache_blocks(new_blocks)

        # NOTE(woosuk): We want to commit (cache) up to num_local_computed_tokens
        # + num_external_computed_tokens + num_new_tokens, but must exclude
        # "non-committable" tokens (e.g., draft tokens that could be rejected).
        # Therefore, we cap the number at `request.num_tokens`, ensuring only
        # "finalized" tokens are cached.
        num_tokens_to_cache = min(
            total_computed_tokens + num_new_tokens,
            request.num_tokens,
        )
        self.coordinator.cache_blocks(request, num_tokens_to_cache)

        return self.create_kv_cache_blocks(new_blocks)
```
**EN:** This method implements `allocate_slots` within `KVCacheManager`. The docstring frames it as: Add slots for a request with new tokens to append. Key calls include `min`, `remove_skipped_blocks`, `get_num_blocks_to_allocate`, `allocate_new_blocks`, `cache_blocks`, `create_kv_cache_blocks`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `allocate_slots`，其作用域位于`KVCacheManager`。 关键调用包括 `min`, `remove_skipped_blocks`, `get_num_blocks_to_allocate`, `allocate_new_blocks`, `cache_blocks`, `create_kv_cache_blocks`。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `KVCacheBlocks`: central class or interface in this module. / `KVCacheBlocks`：本模块中的核心类或接口。
- `KVCacheManager`: central class or interface in this module. / `KVCacheManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `itertools`, `collections`, `dataclasses`, `typing`
- Internal vLLM / 内部依赖: `vllm.distributed.kv_events`, `vllm.logger`, `vllm.v1.core.kv_cache_coordinator`, `vllm.v1.core.kv_cache_metrics`, `vllm.v1.core.kv_cache_utils`, `vllm.v1.kv_cache_interface`, `vllm.v1.metrics.stats`, `vllm.v1.request`
