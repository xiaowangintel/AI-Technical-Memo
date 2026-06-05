# kv_cache_coordinator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/kv_cache_coordinator.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `KVCacheCoordinator`, `KVCacheCoordinatorNoPrefixCache`, `UnitaryKVCacheCoordinator` for the V1 `core` subsystem. / 为 V1 的 `core` 子系统实现 `KVCacheCoordinator`, `KVCacheCoordinatorNoPrefixCache`, `UnitaryKVCacheCoordinator`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from abc import ABC, abstractmethod
from collections.abc import Sequence
from math import lcm

from vllm.v1.core.block_pool import BlockPool
from vllm.v1.core.kv_cache_metrics import KVCacheMetricsCollector
from vllm.v1.core.kv_cache_utils import (
    BlockHash,
    BlockHashList,
    BlockHashListWithBlockSize,
    KVCacheBlock,
)
from vllm.v1.core.single_type_kv_cache_manager import (
    CrossAttentionManager,
    SingleTypeKVCacheManager,
    get_manager_for_kv_cache_spec,
)
from vllm.v1.kv_cache_interface import (
    FullAttentionSpec,
    KVCacheConfig,
    KVCacheSpec,
)
from vllm.v1.request import Request
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.v1.core.block_pool`, `vllm.v1.core.kv_cache_metrics`, `vllm.v1.core.kv_cache_utils`, `vllm.v1.core.single_type_kv_cache_manager`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.v1.core.block_pool`, `vllm.v1.core.kv_cache_metrics`, `vllm.v1.core.kv_cache_utils`, `vllm.v1.core.single_type_kv_cache_manager` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `KVCacheCoordinator` class / `KVCacheCoordinator` 类
```python
class KVCacheCoordinator(ABC):
    """
    Coordinate the KV cache of different KV cache groups.
    """
```
**EN:** Declares the `KVCacheCoordinator` interface. Downstream implementations are expected to provide methods such as `__init__`, `get_num_blocks_to_allocate`, `allocate_new_computed_blocks`, `allocate_new_blocks`, `cache_blocks`, `free`.
**CN:** `KVCacheCoordinator` 声明了一组接口约定。下游实现需要提供 `__init__`, `get_num_blocks_to_allocate`, `allocate_new_computed_blocks`, `allocate_new_blocks`, `cache_blocks`, `free` 等方法。

### `KVCacheCoordinator.__init__` method / `KVCacheCoordinator.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_config: KVCacheConfig,
        max_model_len: int,
        max_num_batched_tokens: int,
        use_eagle: bool,
        enable_caching: bool,
        enable_kv_cache_events: bool,
        dcp_world_size: int,
        pcp_world_size: int,
        hash_block_size: int,
        metrics_collector: KVCacheMetricsCollector | None = None,
    ):
        self.kv_cache_config = kv_cache_config
        self.max_model_len = max_model_len
        self.enable_caching = enable_caching

        self.block_pool = BlockPool(
            num_gpu_blocks=kv_cache_config.num_blocks,
            enable_caching=enable_caching,
            hash_block_size=hash_block_size,
            enable_kv_cache_events=enable_kv_cache_events,
            metrics_collector=metrics_collector,
        )

        # KV cache group indices that get the EAGLE last-block drop.
        self.eagle_group_ids: set[int] = {
            i for i, g in enumerate(kv_cache_config.kv_cache_groups) if g.is_eagle_group
        }
        # Conservatively fall back to flag all groups when no group is flagged.
        if use_eagle and not self.eagle_group_ids:
            self.eagle_group_ids = set(range(len(kv_cache_config.kv_cache_groups)))

        self.single_type_managers = tuple(
            get_manager_for_kv_cache_spec(
                kv_cache_spec=kv_cache_group.kv_cache_spec,
                max_num_batched_tokens=max_num_batched_tokens,
                max_model_len=max_model_len,
                block_pool=self.block_pool,
                enable_caching=enable_caching,
                kv_cache_group_id=i,
                dcp_world_size=dcp_world_size,
                pcp_world_size=pcp_world_size,
            )
            for i, kv_cache_group in enumerate(self.kv_cache_config.kv_cache_groups)
        )
```
**EN:** This method initializes the object state within `KVCacheCoordinator`. Key calls include `BlockPool`, `tuple`, `set`, `enumerate`, `range`, `get_manager_for_kv_cache_spec`. It touches state such as `kv_cache_config`, `max_model_len`, `enable_caching`, `block_pool`, `eagle_group_ids`, `single_type_managers`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`KVCacheCoordinator`。 关键调用包括 `BlockPool`, `tuple`, `set`, `enumerate`, `range`, `get_manager_for_kv_cache_spec`。 它会读写 `kv_cache_config`, `max_model_len`, `enable_caching`, `block_pool`, `eagle_group_ids`, `single_type_managers` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheCoordinator.get_num_blocks_to_allocate` method / `KVCacheCoordinator.get_num_blocks_to_allocate` 方法
```python
    def get_num_blocks_to_allocate(
        self,
        request_id: str,
        num_tokens: int,
        new_computed_blocks: tuple[Sequence[KVCacheBlock], ...],
        num_encoder_tokens: int,
        total_computed_tokens: int,
        num_tokens_main_model: int,
        apply_admission_cap: bool = False,
    ) -> int:
        """
        Get the number of blocks needed to be allocated for the request.

        Args:
            request_id: The request ID.
            num_tokens: The total number of tokens that need a slot (including
                tokens that are already allocated).
            new_computed_blocks: The new computed blocks just hitting the
                prefix caching.
            num_encoder_tokens: The number of encoder tokens for allocating
                blocks for cross-attention.
            total_computed_tokens: Include both local and external tokens.
            num_tokens_main_model: The number of tokens for the main model (aka target
                model in spec decode). w/o spec decode, it is num_tokens;
                with spec decode, it is num_tokens - num_lookahead_tokens.
            apply_admission_cap: If True, apply the recycling-aware
                per-request admission cap (SWA / chunked-local). Set only by
                the full-sequence admission gate; per-step allocation must
                leave it False so the predictor matches `allocate_new_blocks`.

        Returns:
            The number of blocks to allocate.
        """
        num_blocks_to_allocate = 0
        for i, manager in enumerate(self.single_type_managers):
            if isinstance(manager, CrossAttentionManager):
                # For cross-attention, we issue a single static allocation
                # of blocks based on the number of encoder input tokens.
                num_blocks_to_allocate += manager.get_num_blocks_to_allocate(
                    request_id,
                    num_encoder_tokens,
                    [],
                    0,
                    num_encoder_tokens,
                    apply_admission_cap=apply_admission_cap,
                )
            else:
                num_blocks_to_allocate += manager.get_num_blocks_to_allocate(
                    request_id,
                    num_tokens,
                    new_computed_blocks[i],
                    total_computed_tokens,
                    num_tokens_main_model,
                    apply_admission_cap=apply_admission_cap,
                )
        return num_blocks_to_allocate
```
**EN:** This method returns or derives a value within `KVCacheCoordinator`. The docstring frames it as: Get the number of blocks needed to be allocated for the request. Key calls include `enumerate`, `isinstance`, `get_num_blocks_to_allocate`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`KVCacheCoordinator`。 关键调用包括 `enumerate`, `isinstance`, `get_num_blocks_to_allocate`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheCoordinator.allocate_new_computed_blocks` method / `KVCacheCoordinator.allocate_new_computed_blocks` 方法
```python
    def allocate_new_computed_blocks(
        self,
        request_id: str,
        new_computed_blocks: tuple[Sequence[KVCacheBlock], ...],
        num_local_computed_tokens: int,
        num_external_computed_tokens: int,
    ) -> None:
        """
        Add the new computed blocks to the request. Optionally allocate new
            blocks for external computed tokens (if any).

        Args:
            request_id: The request ID.
            new_computed_blocks: The new computed blocks just hitting the
                prefix cache.
            num_local_computed_tokens: The number of local computed tokens.
            num_external_computed_tokens: The number of external computed tokens.
        """
        for i, manager in enumerate(self.single_type_managers):
            manager.allocate_new_computed_blocks(
                request_id,
                new_computed_blocks[i],
                num_local_computed_tokens,
                num_external_computed_tokens,
            )
```
**EN:** This method implements `allocate_new_computed_blocks` within `KVCacheCoordinator`. The docstring frames it as: Add the new computed blocks to the request. Key calls include `enumerate`, `allocate_new_computed_blocks`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `allocate_new_computed_blocks`，其作用域位于`KVCacheCoordinator`。 关键调用包括 `enumerate`, `allocate_new_computed_blocks`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `KVCacheCoordinatorNoPrefixCache` class / `KVCacheCoordinatorNoPrefixCache` 类
```python
class KVCacheCoordinatorNoPrefixCache(KVCacheCoordinator):
    """
    KV cache coordinator to use if prefix caching is disabled or unsupported.
    In contrast to UnitaryKVCacheCoordinator and HybridKVCacheCoordinator,
    supports arbitrary numbers of KV cache groups (including 0 groups).
    Does not implement any features related to prefix caching.
    """
```
**EN:** Introduces the `KVCacheCoordinatorNoPrefixCache` class on top of `KVCacheCoordinator`. Core methods include `__init__`, `get_num_common_prefix_blocks`, `find_longest_cache_hit`. Docstring signal: KV cache coordinator to use if prefix caching is disabled or unsupported.
**CN:** 这里定义 `KVCacheCoordinatorNoPrefixCache` 类，其基类包括 `KVCacheCoordinator`。核心方法包括 `__init__`, `get_num_common_prefix_blocks`, `find_longest_cache_hit`。

### `KVCacheCoordinatorNoPrefixCache.__init__` method / `KVCacheCoordinatorNoPrefixCache.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_config: KVCacheConfig,
        max_model_len: int,
        max_num_batched_tokens: int,
        use_eagle: bool,
        enable_kv_cache_events: bool,
        dcp_world_size: int,
        pcp_world_size: int,
        hash_block_size: int,
        metrics_collector: KVCacheMetricsCollector | None = None,
    ):
        super().__init__(
            kv_cache_config,
            max_model_len,
            max_num_batched_tokens,
            use_eagle,
            False,
            enable_kv_cache_events,
            dcp_world_size=dcp_world_size,
            pcp_world_size=pcp_world_size,
            hash_block_size=hash_block_size,
            metrics_collector=metrics_collector,
        )
        self.num_single_type_manager = len(self.single_type_managers)
```
**EN:** This method initializes the object state within `KVCacheCoordinatorNoPrefixCache`. Key calls include `__init__`, `len`, `super`. It touches state such as `num_single_type_manager`.
**CN:** 该方法会初始化对象状态，其作用域位于`KVCacheCoordinatorNoPrefixCache`。 关键调用包括 `__init__`, `len`, `super`。 它会读写 `num_single_type_manager` 等状态。

### `KVCacheCoordinatorNoPrefixCache.get_num_common_prefix_blocks` method / `KVCacheCoordinatorNoPrefixCache.get_num_common_prefix_blocks` 方法
```python
    def get_num_common_prefix_blocks(self, running_request_id: str) -> list[int]:
        return [0] * self.num_single_type_manager
```
**EN:** This method returns or derives a value within `KVCacheCoordinatorNoPrefixCache`.
**CN:** 该方法会返回或推导一个值，其作用域位于`KVCacheCoordinatorNoPrefixCache`。

### `KVCacheCoordinatorNoPrefixCache.find_longest_cache_hit` method / `KVCacheCoordinatorNoPrefixCache.find_longest_cache_hit` 方法
```python
    def find_longest_cache_hit(
        self,
        block_hashes: list[BlockHash],
        max_cache_hit_length: int,
    ) -> tuple[tuple[list[KVCacheBlock], ...], int]:
        blocks: tuple[list[KVCacheBlock], ...] = tuple(
            [] for _ in range(self.num_single_type_manager)
        )
        return blocks, 0
```
**EN:** This method implements `find_longest_cache_hit` within `KVCacheCoordinatorNoPrefixCache`. Key calls include `tuple`, `range`.
**CN:** 该方法会实现 `find_longest_cache_hit`，其作用域位于`KVCacheCoordinatorNoPrefixCache`。 关键调用包括 `tuple`, `range`。

### `UnitaryKVCacheCoordinator` class / `UnitaryKVCacheCoordinator` 类
```python
class UnitaryKVCacheCoordinator(KVCacheCoordinator):
    """
    KV cache coordinator for models with only one KV cache group. This is the
    case for models with only one KV cache type, e.g., all attention layers use
    full attention or all attention layers use sliding window attention.
    """
```
**EN:** Introduces the `UnitaryKVCacheCoordinator` class on top of `KVCacheCoordinator`. Core methods include `__init__`, `find_longest_cache_hit`. Docstring signal: KV cache coordinator for models with only one KV cache group.
**CN:** 这里定义 `UnitaryKVCacheCoordinator` 类，其基类包括 `KVCacheCoordinator`。核心方法包括 `__init__`, `find_longest_cache_hit`。

### `UnitaryKVCacheCoordinator.__init__` method / `UnitaryKVCacheCoordinator.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_config: KVCacheConfig,
        max_model_len: int,
        max_num_batched_tokens: int,
        use_eagle: bool,
        enable_caching: bool,
        enable_kv_cache_events: bool,
        dcp_world_size: int,
        pcp_world_size: int,
        hash_block_size: int,
        metrics_collector: KVCacheMetricsCollector | None = None,
    ):
        super().__init__(
            kv_cache_config,
            max_model_len,
            max_num_batched_tokens,
            use_eagle,
            enable_caching,
            enable_kv_cache_events,
            dcp_world_size=dcp_world_size,
            pcp_world_size=pcp_world_size,
            hash_block_size=hash_block_size,
            metrics_collector=metrics_collector,
        )
        self.kv_cache_spec = self.kv_cache_config.kv_cache_groups[0].kv_cache_spec
        self.block_size = self.kv_cache_spec.block_size
        self.dcp_world_size = dcp_world_size
        self.pcp_world_size = pcp_world_size
        if dcp_world_size > 1:
            self.block_size *= dcp_world_size
        if pcp_world_size > 1:
            self.block_size *= pcp_world_size
        # For models using only Mamba, block_size is set to max_model_len when
        # prefix caching is disabled, and hash_block_size validation is skipped.
        assert not enable_caching or (hash_block_size == self.block_size), (
            "UnitaryKVCacheCoordinator assumes hash_block_size == block_size"
        )
        assert len(self.kv_cache_config.kv_cache_groups) == 1, (
            "UnitaryKVCacheCoordinator assumes only one kv cache group"
        )
```
**EN:** This method initializes the object state within `UnitaryKVCacheCoordinator`. Key calls include `__init__`, `len`, `super`. It touches state such as `kv_cache_spec`, `block_size`, `dcp_world_size`, `pcp_world_size`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`UnitaryKVCacheCoordinator`。 关键调用包括 `__init__`, `len`, `super`。 它会读写 `kv_cache_spec`, `block_size`, `dcp_world_size`, `pcp_world_size` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UnitaryKVCacheCoordinator.find_longest_cache_hit` method / `UnitaryKVCacheCoordinator.find_longest_cache_hit` 方法
```python
    def find_longest_cache_hit(
        self,
        block_hashes: list[BlockHash],
        max_cache_hit_length: int,
    ) -> tuple[tuple[list[KVCacheBlock], ...], int]:
        hit_blocks = self.single_type_managers[0].find_longest_cache_hit(
            block_hashes=block_hashes,
            max_length=max_cache_hit_length,
            kv_cache_group_ids=[0],
            block_pool=self.block_pool,
            kv_cache_spec=self.kv_cache_spec,
            use_eagle=0 in self.eagle_group_ids,
            alignment_tokens=self.block_size,
            dcp_world_size=self.dcp_world_size,
            pcp_world_size=self.pcp_world_size,
        )
        return hit_blocks, len(hit_blocks[0]) * self.block_size
```
**EN:** This method implements `find_longest_cache_hit` within `UnitaryKVCacheCoordinator`. Key calls include `find_longest_cache_hit`, `len`.
**CN:** 该方法会实现 `find_longest_cache_hit`，其作用域位于`UnitaryKVCacheCoordinator`。 关键调用包括 `find_longest_cache_hit`, `len`。

### `HybridKVCacheCoordinator` class / `HybridKVCacheCoordinator` 类
```python
class HybridKVCacheCoordinator(KVCacheCoordinator):
    """
    KV cache coordinator for hybrid models with multiple KV cache types, and
    thus multiple kv cache groups.
    """
```
**EN:** Introduces the `HybridKVCacheCoordinator` class on top of `KVCacheCoordinator`. Core methods include `__init__`, `verify_and_split_kv_cache_groups`, `cache_blocks`, `find_longest_cache_hit`. Docstring signal: KV cache coordinator for hybrid models with multiple KV cache types, and thus multiple kv cache groups.
**CN:** 这里定义 `HybridKVCacheCoordinator` 类，其基类包括 `KVCacheCoordinator`。核心方法包括 `__init__`, `verify_and_split_kv_cache_groups`, `cache_blocks`, `find_longest_cache_hit`。

### `HybridKVCacheCoordinator.__init__` method / `HybridKVCacheCoordinator.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_config: KVCacheConfig,
        max_model_len: int,
        max_num_batched_tokens: int,
        use_eagle: bool,
        enable_caching: bool,
        enable_kv_cache_events: bool,
        dcp_world_size: int,
        pcp_world_size: int,
        hash_block_size: int,
        metrics_collector: KVCacheMetricsCollector | None = None,
    ):
        super().__init__(
            kv_cache_config,
            max_model_len,
            max_num_batched_tokens,
            use_eagle,
            enable_caching,
            enable_kv_cache_events,
            dcp_world_size=dcp_world_size,
            pcp_world_size=pcp_world_size,
            hash_block_size=hash_block_size,
            metrics_collector=metrics_collector,
        )
        # hash_block_size: the block size used to compute block hashes.
        # The actual block size usually equals hash_block_size, but in cases where
        # different KV cache groups have different block sizes, the actual block size
        # can be a multiple of hash_block_size.
        self.hash_block_size = hash_block_size
        assert all(
            g.kv_cache_spec.block_size % hash_block_size == 0
            for g in kv_cache_config.kv_cache_groups
        ), "block_size must be divisible by hash_block_size"
        assert dcp_world_size == 1, "DCP not support hybrid attn now."
        assert pcp_world_size == 1, "PCP not support hybrid attn now."
        self.verify_and_split_kv_cache_groups()
```
**EN:** This method initializes the object state within `HybridKVCacheCoordinator`. Key calls include `__init__`, `all`, `verify_and_split_kv_cache_groups`, `super`. It touches state such as `hash_block_size`.
**CN:** 该方法会初始化对象状态，其作用域位于`HybridKVCacheCoordinator`。 关键调用包括 `__init__`, `all`, `verify_and_split_kv_cache_groups`, `super`。 它会读写 `hash_block_size` 等状态。

### `HybridKVCacheCoordinator.verify_and_split_kv_cache_groups` method / `HybridKVCacheCoordinator.verify_and_split_kv_cache_groups` 方法
```python
    def verify_and_split_kv_cache_groups(self) -> None:
        """
        Groups KV cache groups by their spec type for efficient batch processing
        during cache hit lookup.
        """
        attention_groups: list[
            tuple[KVCacheSpec, list[int], type[SingleTypeKVCacheManager]]
        ] = []

        for i, g in enumerate(self.kv_cache_config.kv_cache_groups):
            manager_cls = self.single_type_managers[i].__class__
            spec = g.kv_cache_spec

            # Try to find an existing group with the same spec
            for existing_spec, group_ids, existing_cls in attention_groups:
                if existing_spec == spec:
                    assert manager_cls is existing_cls, (
                        "Expected same manager class for identical KV cache specs."
                    )
                    group_ids.append(i)
                    break
            else:
                attention_groups.append((spec, [i], manager_cls))

        assert len(attention_groups) > 1, (
            "HybridKVCacheCoordinator requires at least two attention groups."
        )

        # Put full attention first: its efficient left-to-right scan provides
        # a tighter initial bound, reducing work for subsequent groups.
        self.attention_groups = sorted(
            attention_groups,
            key=lambda x: not isinstance(x[0], FullAttentionSpec),
        )

        # The LCM of the block sizes of all attention types.
        # The cache hit length must be a multiple of the LCM of the block sizes
        # to make sure the cache hit length is a multiple of the block size of
        # each attention type. Requiring this because we don't support partial
        # block cache hit yet.
        block_sizes = [spec.block_size for spec, _, _ in attention_groups]
        self.lcm_block_size = lcm(*block_sizes)

        # Attention-group indices (into ``self.attention_groups``) that
        # contain at least one EAGLE/MTP KV cache group.
        self.eagle_attn_group_indices: set[int] = {
            i
            for i, (_, group_ids, _) in enumerate(self.attention_groups)
            if any(gid in self.eagle_group_ids for gid in group_ids)
        }
```
**EN:** This method implements `verify_and_split_kv_cache_groups` within `HybridKVCacheCoordinator`. The docstring frames it as: Groups KV cache groups by their spec type for efficient batch processing during cache hit lookup. Key calls include `enumerate`, `sorted`, `lcm`, `len`, `append`, `any`. It touches state such as `attention_groups`, `lcm_block_size`, `eagle_attn_group_indices`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `verify_and_split_kv_cache_groups`，其作用域位于`HybridKVCacheCoordinator`。 关键调用包括 `enumerate`, `sorted`, `lcm`, `len`, `append`, `any`。 它会读写 `attention_groups`, `lcm_block_size`, `eagle_attn_group_indices` 等状态。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `HybridKVCacheCoordinator.find_longest_cache_hit` method / `HybridKVCacheCoordinator.find_longest_cache_hit` 方法
```python
    def find_longest_cache_hit(
        self,
        block_hashes: list[BlockHash],
        max_cache_hit_length: int,
    ) -> tuple[tuple[list[KVCacheBlock], ...], int]:
        """
        Find the longest cache hit using an iterative fixed-point algorithm.

        Each attention type either accepts the current candidate length or
        reduces it. If any type reduces the length, restart checks over all
        types. This converges because length monotonically decreases and is
        bounded below by 0.

        Args:
            block_hashes: The block hashes of the request.
            max_cache_hit_length: The maximum length of the cache hit.

        Returns:
            A tuple containing:
                - A tuple of the cache hit blocks for each single type manager.
                - The number of tokens of the longest cache hit.
        """

        def _get_block_hashes(kv_cache_spec: KVCacheSpec) -> BlockHashList:
            if kv_cache_spec.block_size == self.hash_block_size:
                return block_hashes
            return BlockHashListWithBlockSize(
                block_hashes, self.hash_block_size, kv_cache_spec.block_size
            )

        num_groups = len(self.kv_cache_config.kv_cache_groups)
        hit_length = max_cache_hit_length
        hit_blocks_by_group: list[list[KVCacheBlock] | None] = [None] * num_groups

        # Simple hybrid (1 full attn + 1 other): one iteration suffices.
        # Full attn is always first if it exists.
        is_simple_hybrid = len(self.attention_groups) == 2 and isinstance(
            self.attention_groups[0][0], FullAttentionSpec
        )

        # Attention-group indices whose EAGLE drop is verified at the current
        # ``curr_hit_length``. Each eagle group applies the drop at most once
        # per candidate length (see issue #32802).
        eagle_verified: set[int] = set()

        while True:
            curr_hit_length = hit_length

            for idx, (spec, group_ids, manager_cls) in enumerate(self.attention_groups):
                cached_blocks = hit_blocks_by_group[group_ids[0]]
    # ... omitted for brevity ...
                    eagle_verified.add(idx)
                elif _new_hit_length < curr_hit_length:
                    # length shrunk; invalidate previous eagle verifications
                    eagle_verified.clear()
                curr_hit_length = _new_hit_length
                for group_id, blocks in zip(group_ids, hit_blocks):
                    hit_blocks_by_group[group_id] = blocks

            if curr_hit_length >= hit_length:
                break
            hit_length = curr_hit_length
            if is_simple_hybrid:
                break

        # Truncate full attention blocks to final hit_length (if present)
        spec, group_ids, _ = self.attention_groups[0]
        if isinstance(spec, FullAttentionSpec):
            num_blocks = hit_length // spec.block_size
            for group_id in group_ids:
                if (blks := hit_blocks_by_group[group_id]) is not None:
                    del blks[num_blocks:]

        return tuple(
            blocks if blocks is not None else [] for blocks in hit_blocks_by_group
        ), hit_length
```
**EN:** This method implements `find_longest_cache_hit` within `HybridKVCacheCoordinator`. The docstring frames it as: Find the longest cache hit using an iterative fixed-point algorithm. Key calls include `len`, `set`, `isinstance`, `BlockHashListWithBlockSize`, `enumerate`, `tuple`. The control flow contains 10 branch(es) and 4 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `find_longest_cache_hit`，其作用域位于`HybridKVCacheCoordinator`。 关键调用包括 `len`, `set`, `isinstance`, `BlockHashListWithBlockSize`, `enumerate`, `tuple`。 控制流包含 10 个分支和 4 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `get_kv_cache_coordinator` function / `get_kv_cache_coordinator` 函数
```python
def get_kv_cache_coordinator(
    kv_cache_config: KVCacheConfig,
    max_model_len: int,
    max_num_batched_tokens: int,
    use_eagle: bool,
    enable_caching: bool,
    enable_kv_cache_events: bool,
    dcp_world_size: int,
    pcp_world_size: int,
    hash_block_size: int,
    metrics_collector: KVCacheMetricsCollector | None = None,
) -> KVCacheCoordinator:
    if not enable_caching:
        return KVCacheCoordinatorNoPrefixCache(
            kv_cache_config,
            max_model_len,
            max_num_batched_tokens,
            use_eagle,
            enable_kv_cache_events,
            dcp_world_size=dcp_world_size,
            pcp_world_size=pcp_world_size,
            hash_block_size=hash_block_size,
            metrics_collector=metrics_collector,
        )
    if len(kv_cache_config.kv_cache_groups) == 1:
        return UnitaryKVCacheCoordinator(
            kv_cache_config,
            max_model_len,
            max_num_batched_tokens,
            use_eagle,
            enable_caching,
            enable_kv_cache_events,
            dcp_world_size=dcp_world_size,
            pcp_world_size=pcp_world_size,
            hash_block_size=hash_block_size,
            metrics_collector=metrics_collector,
        )
    return HybridKVCacheCoordinator(
        kv_cache_config,
        max_model_len,
        max_num_batched_tokens,
        use_eagle,
        enable_caching,
        enable_kv_cache_events,
        dcp_world_size=dcp_world_size,
        pcp_world_size=pcp_world_size,
        hash_block_size=hash_block_size,
        metrics_collector=metrics_collector,
    )
```
**EN:** This function returns or derives a value within the module. Key calls include `HybridKVCacheCoordinator`, `KVCacheCoordinatorNoPrefixCache`, `len`, `UnitaryKVCacheCoordinator`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `HybridKVCacheCoordinator`, `KVCacheCoordinatorNoPrefixCache`, `len`, `UnitaryKVCacheCoordinator`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `KVCacheCoordinator`: central class or interface in this module. / `KVCacheCoordinator`：本模块中的核心类或接口。
- `KVCacheCoordinatorNoPrefixCache`: central class or interface in this module. / `KVCacheCoordinatorNoPrefixCache`：本模块中的核心类或接口。
- `UnitaryKVCacheCoordinator`: central class or interface in this module. / `UnitaryKVCacheCoordinator`：本模块中的核心类或接口。
- `HybridKVCacheCoordinator`: central class or interface in this module. / `HybridKVCacheCoordinator`：本模块中的核心类或接口。
- `get_kv_cache_coordinator`: top-level helper or orchestration entry point. / `get_kv_cache_coordinator`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `abc`, `collections`, `math`
- Internal vLLM / 内部依赖: `vllm.v1.core.block_pool`, `vllm.v1.core.kv_cache_metrics`, `vllm.v1.core.kv_cache_utils`, `vllm.v1.core.single_type_kv_cache_manager`, `vllm.v1.kv_cache_interface`, `vllm.v1.request`
