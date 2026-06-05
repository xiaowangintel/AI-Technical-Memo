# kv_cache_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/kv_cache_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: KV-Cache Utilities. / 该模块位于 `core` 子系统，主要围绕 `make_block_hash_with_group_id`, `get_block_hash`, `get_group_id` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""KV-Cache Utilities."""

import copy
import hashlib
import math
import os
from collections import defaultdict
from collections.abc import Callable, Iterable, Iterator, Sequence
from dataclasses import dataclass, replace
from functools import partial
from typing import Any, NewType, TypeAlias, cast, overload

from vllm import envs
from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.utils.hashing import sha256_cbor, xxhash_cbor
from vllm.utils.math_utils import cdiv, round_up
from vllm.utils.mem_utils import format_gib
from vllm.utils.torch_utils import get_dtype_size
from vllm.v1.kv_cache_interface import (
    ChunkedLocalAttentionSpec,
    FullAttentionSpec,
    HiddenStateCacheSpec,
    KVCacheConfig,
    KVCacheGroupSpec,
    KVCacheSpec,
    KVCacheTensor,
    MambaSpec,
    MLAAttentionSpec,
    SlidingWindowMLASpec,
    SlidingWindowSpec,
    UniformTypeKVCacheSpecs,
)
from vllm.v1.request import Request
from vllm.v1.utils import tensor_data

# BlockHash represents the hash of a single KV-cache block used for
# prefix caching.  Treating it as a distinct type from `bytes` helps
# catch accidental misuse when passing around raw byte strings.
BlockHash = NewType("BlockHash", bytes)

# `BlockHashWithGroupId` combines a `BlockHash` with its KV cache group ID.
# It is represented as raw bytes for compactness and efficiency. The helper
# functions below pack/unpack the `BlockHash` and group id into/from the key.
BlockHashWithGroupId = NewType("BlockHashWithGroupId", bytes)

# ExternalBlockHash is used for reproducible prefix-cache block hashing.
# It's a union of `bytes` and `int` to keep backward compatibility
# after we default block hashing to use sha256 bytes.
ExternalBlockHash: TypeAlias = bytes | int
# ... omitted for brevity ...


def get_group_id(key: BlockHashWithGroupId) -> int:
    """Extract the group id from a `BlockHashWithGroupId`."""
    return int.from_bytes(key[-4:], "big", signed=False)


def maybe_convert_block_hash(hash_bytes: BlockHash) -> ExternalBlockHash:
    if not envs.VLLM_KV_EVENTS_USE_INT_BLOCK_HASHES:
        return hash_bytes
    return int.from_bytes(hash_bytes, byteorder="big") & ((1 << 64) - 1)


logger = init_logger(__name__)

# The hash seed for the first block of any prefix block sequence.
#
# We use a random value to avoid hash collisions or PYTHONHASHSEED environment
# variable if set such that processes can share the seed if needed. This aligns
# with the behavior of Python's hash() function, which also uses a random seed
# if PYTHONHASHSEED is not set.
#
# The function `init_none_hash` initializes this variable globally.
NONE_HASH: BlockHash
_CBOR_HASH_FUNCTIONS = frozenset({sha256_cbor, xxhash_cbor})
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `BlockHash`, `BlockHashWithGroupId`, `ExternalBlockHash`, `logger`, `NONE_HASH`, `_CBOR_HASH_FUNCTIONS`. This excerpt omits repetitive declarations.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `BlockHash`, `BlockHashWithGroupId`, `ExternalBlockHash`, `logger`, `NONE_HASH`, `_CBOR_HASH_FUNCTIONS`。 该片段省略了重复性声明。

### `KVCacheBlock` class / `KVCacheBlock` 类
```python
@dataclass(slots=True)
class KVCacheBlock:
    """KV-cache block metadata."""

    # Block ID, ranging from 0 to num_gpu_blocks - 1.
    block_id: int
    # Reference count.
    ref_cnt: int = 0
    # The hash key (block hash + group id) of the block, only available
    # when the block is full and cached.
    _block_hash: BlockHashWithGroupId | None = None

    # Used to construct a doubly linked list for free blocks.
    # These two attributes should only be manipulated by FreeKVCacheBlockQueue.
    prev_free_block: "KVCacheBlock | None" = None
    next_free_block: "KVCacheBlock | None" = None

    # Whether the block is a null block that should never be cached.
    is_null: bool = False
```
**EN:** Introduces the `KVCacheBlock` class. Core methods include `block_hash`, `block_hash`, `reset_hash`, `__repr__`. Docstring signal: KV-cache block metadata.
**CN:** 这里定义 `KVCacheBlock` 类。核心方法包括 `block_hash`, `block_hash`, `reset_hash`, `__repr__`。

### `KVCacheBlock.block_hash` method / `KVCacheBlock.block_hash` 方法
```python
    @property
    def block_hash(self) -> BlockHashWithGroupId | None:
        return self._block_hash
```
**EN:** This method implements `block_hash` within `KVCacheBlock`.
**CN:** 该方法会实现 `block_hash`，其作用域位于`KVCacheBlock`。

### `KVCacheBlock.block_hash` method / `KVCacheBlock.block_hash` 方法
```python
    @block_hash.setter
    def block_hash(self, block_hash: BlockHashWithGroupId):
        assert self.block_hash is None, (
            "The block already has a hash. This should not happen."
        )
        self._block_hash = block_hash
```
**EN:** This method implements `block_hash` within `KVCacheBlock`. It touches state such as `_block_hash`.
**CN:** 该方法会实现 `block_hash`，其作用域位于`KVCacheBlock`。 它会读写 `_block_hash` 等状态。

### `KVCacheBlock.reset_hash` method / `KVCacheBlock.reset_hash` 方法
```python
    def reset_hash(self):
        """Reset the block hash when the block is evicted."""
        self._block_hash = None
```
**EN:** This method implements `reset_hash` within `KVCacheBlock`. The docstring frames it as: Reset the block hash when the block is evicted. It touches state such as `_block_hash`.
**CN:** 该方法会实现 `reset_hash`，其作用域位于`KVCacheBlock`。 它会读写 `_block_hash` 等状态。

### `KVCacheBlock.__repr__` method / `KVCacheBlock.__repr__` 方法
```python
    def __repr__(self) -> str:
        # Use block_id instead of KVCacheBlock object to avoid calling __repr__
        # on KVCacheBlock object recursively.
        prev_block_id = self.prev_free_block.block_id if self.prev_free_block else None
        next_block_id = self.next_free_block.block_id if self.next_free_block else None
        return (
            f"KVCacheBlock(block_id={self.block_id}, "
            f"ref_cnt={self.ref_cnt}, "
            f"_block_hash={self._block_hash!r}, "
            f"prev_free_block={prev_block_id}, "
            f"next_free_block={next_block_id})"
        )
```
**EN:** This method implements `__repr__` within `KVCacheBlock`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__repr__`，其作用域位于`KVCacheBlock`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FreeKVCacheBlockQueue` class / `FreeKVCacheBlockQueue` 类
```python
class FreeKVCacheBlockQueue:
    """This class organizes a list of KVCacheBlock objects to a doubly linked
    list of free blocks. We implement this class instead of using Python
    builtin deque to support removing a block in the middle of the queue
    in O(1) time. To close the performance gap to the builtin deque which is
    implemented in C++, this class does not allocate any Python objects when
    manipulating the linked list. Instead, this class manipulates the
    prev_free_block and next_free_block attributes of the given blocks.

    The queue is ordered by block ID in the beginning. When a block is allocated
    and then freed, it will be appended back with the eviction order:
    1. The least recent used block is at the front (LRU).
    2. If two blocks have the same last accessed time (allocated by the
       same sequence), the one with more hash tokens (the tail of a block
       chain) is at the front.
    Note that we maintain this order by reversing the block order when free
    blocks of a request. This operation is outside of this class.

    Args:
        blocks: A list of KVCacheBlock objects.
    """
```
**EN:** Introduces the `FreeKVCacheBlockQueue` class. Core methods include `__init__`, `popleft`, `popleft_n`, `remove`, `append`, `append_n`. Docstring signal: This class organizes a list of KVCacheBlock objects to a doubly linked list of free blocks.
**CN:** 这里定义 `FreeKVCacheBlockQueue` 类。核心方法包括 `__init__`, `popleft`, `popleft_n`, `remove`, `append`, `append_n`。

### `FreeKVCacheBlockQueue.__init__` method / `FreeKVCacheBlockQueue.__init__` 方法
```python
    def __init__(self, blocks: list[KVCacheBlock]) -> None:
        self.num_free_blocks = len(blocks)

        # Initialize doubly links of consecutive blocks
        for i in range(self.num_free_blocks):
            if i > 0:
                blocks[i].prev_free_block = blocks[i - 1]
            if i < self.num_free_blocks - 1:
                blocks[i].next_free_block = blocks[i + 1]

        # Create a fake head and a tail block for the doubly linked list to
        # reduce branching in the code
        #
        # The implementation guaranteed that the fake head and tail
        # are NEVER got popped, so we could safely assume each real blocks
        # in the queue has prev and next blocks.
        self.fake_free_list_head = KVCacheBlock(block_id=-1)
        self.fake_free_list_tail = KVCacheBlock(block_id=-1)
        if self.num_free_blocks > 0:
            # Connect fake_head and fake_tail to the first and last block
            # respectively.
            self.fake_free_list_head.next_free_block = blocks[0]
            blocks[0].prev_free_block = self.fake_free_list_head
            self.fake_free_list_tail.prev_free_block = blocks[-1]
            blocks[-1].next_free_block = self.fake_free_list_tail
        else:
            # For empty list, simply connect the fake head and tail.
            self.fake_free_list_head.next_free_block = self.fake_free_list_tail
            self.fake_free_list_tail.prev_free_block = self.fake_free_list_head
```
**EN:** This method initializes the object state within `FreeKVCacheBlockQueue`. Key calls include `len`, `range`, `KVCacheBlock`. It touches state such as `num_free_blocks`, `fake_free_list_head`, `fake_free_list_tail`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`FreeKVCacheBlockQueue`。 关键调用包括 `len`, `range`, `KVCacheBlock`。 它会读写 `num_free_blocks`, `fake_free_list_head`, `fake_free_list_tail` 等状态。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `FreeKVCacheBlockQueue.popleft` method / `FreeKVCacheBlockQueue.popleft` 方法
```python
    def popleft(self) -> KVCacheBlock:
        """Pop the first free block and reduce num_free_blocks by 1.

        Returns:
            The first free block.
        """
        if (
            self.fake_free_list_head.next_free_block is self.fake_free_list_tail
            or self.fake_free_list_head.next_free_block is None
        ):
            assert self.num_free_blocks == 0, (
                f"num_free_blocks ({self.num_free_blocks}) is out of sync "
                "with the free list."
            )
            raise ValueError("No free blocks available")

        first_block: KVCacheBlock = self.fake_free_list_head.next_free_block

        if first_block.next_free_block is None:
            # This should not happen if the block is from the free list.
            # It indicates a bug in the caller's logic.
            raise RuntimeError(
                "Invalid block found in popleft() "
                "which doesn't have a valid next_free_block"
            )

        # Connect fake_head and the next block of first_block (i.e. second block
        # or fake tail).
        self.fake_free_list_head.next_free_block = first_block.next_free_block
        first_block.next_free_block.prev_free_block = self.fake_free_list_head

        # Remove the block from the linked list.
        first_block.prev_free_block = first_block.next_free_block = None

        self.num_free_blocks -= 1
        return first_block
```
**EN:** This method implements `popleft` within `FreeKVCacheBlockQueue`. The docstring frames it as: Pop the first free block and reduce num_free_blocks by 1. Key calls include `ValueError`, `RuntimeError`. It touches state such as `fake_free_list_head`, `num_free_blocks`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `popleft`，其作用域位于`FreeKVCacheBlockQueue`。 关键调用包括 `ValueError`, `RuntimeError`。 它会读写 `fake_free_list_head`, `num_free_blocks` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FreeKVCacheBlockQueue.popleft_n` method / `FreeKVCacheBlockQueue.popleft_n` 方法
```python
    def popleft_n(self, n: int) -> list[KVCacheBlock]:
        """Pop the first n free blocks and reduce num_free_blocks by n.

        Args:
            n: The number of blocks to pop.

        Returns:
            A list of n free blocks.
        """
        if n == 0:
            return []
        assert self.num_free_blocks >= n
        self.num_free_blocks -= n

        curr_block = self.fake_free_list_head.next_free_block
        # Pop n blocks from the head of the list
        ret = []
        for _ in range(n):
            assert curr_block is not None
            ret.append(curr_block)
            last_block = curr_block
            curr_block = curr_block.next_free_block
            # Reset prev_free_block and next_free_block of all popped blocks
            last_block.prev_free_block = None
            last_block.next_free_block = None

        if curr_block is not None:
            # The queue is not empty, connect the fake head to
            # the new first block.
            self.fake_free_list_head.next_free_block = curr_block
            curr_block.prev_free_block = self.fake_free_list_head
        return ret
```
**EN:** This method implements `popleft_n` within `FreeKVCacheBlockQueue`. The docstring frames it as: Pop the first n free blocks and reduce num_free_blocks by n. Key calls include `range`, `append`. It touches state such as `num_free_blocks`, `fake_free_list_head`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `popleft_n`，其作用域位于`FreeKVCacheBlockQueue`。 关键调用包括 `range`, `append`。 它会读写 `num_free_blocks`, `fake_free_list_head` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `FreeKVCacheBlockQueue.append_n` method / `FreeKVCacheBlockQueue.append_n` 方法
```python
    def append_n(self, blocks: list[KVCacheBlock]) -> None:
        """Put a list of blocks back into the free list

        Args:
            blocks: The blocks to append.
        """
        if len(blocks) == 0:
            return

        last_block = self.fake_free_list_tail.prev_free_block
        assert last_block is not None, (
            "prev_free_block of fake_free_list_tail should always exist"
        )
        # Add inter-connections between consecutive blocks
        for block in blocks:
            block.prev_free_block = last_block
            last_block.next_free_block = block
            last_block = block

        # Connect the last block of <blocks> to the fake tail
        last_block.next_free_block = self.fake_free_list_tail
        self.fake_free_list_tail.prev_free_block = last_block

        self.num_free_blocks += len(blocks)
```
**EN:** This method implements `append_n` within `FreeKVCacheBlockQueue`. The docstring frames it as: Put a list of blocks back into the free list Args: blocks: The blocks to append. Key calls include `len`. It touches state such as `fake_free_list_tail`, `num_free_blocks`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `append_n`，其作用域位于`FreeKVCacheBlockQueue`。 关键调用包括 `len`。 它会读写 `fake_free_list_tail`, `num_free_blocks` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `FreeKVCacheBlockQueue.get_all_free_blocks` method / `FreeKVCacheBlockQueue.get_all_free_blocks` 方法
```python
    def get_all_free_blocks(self) -> list[KVCacheBlock]:
        """Get all free blocks in the free list. Mainly used for testing.

        Returns:
            A list of free blocks.
        """
        ret = []
        if self.fake_free_list_head.next_free_block is None:
            raise RuntimeError(
                "next_free_block of fake_free_list_head should always exist"
            )
        # Start from the first block
        curr_block: KVCacheBlock = self.fake_free_list_head.next_free_block
        # As long as next_free_block is available, we haven't reached to
        # the fake tail yet.
        while curr_block.next_free_block is not None:
            ret.append(curr_block)
            curr_block = curr_block.next_free_block
        return ret
```
**EN:** This method returns or derives a value within `FreeKVCacheBlockQueue`. The docstring frames it as: Get all free blocks in the free list. Key calls include `RuntimeError`, `append`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`FreeKVCacheBlockQueue`。 关键调用包括 `RuntimeError`, `append`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `resolve_kv_cache_block_sizes` function / `resolve_kv_cache_block_sizes` 函数
```python
def resolve_kv_cache_block_sizes(
    kv_cache_config: KVCacheConfig,
    vllm_config: VllmConfig,
) -> tuple[int, int]:
    """Resolve (scheduler_block_size, hash_block_size).

    - ``scheduler_block_size`` is the token-alignment invariant used by the
      scheduler (e.g. for ``num_computed_tokens`` rounding). Single group:
      ``cache_config.block_size * dcp * pcp``. Multiple groups: LCM of every
      group's block size — context parallelism is not supported here.
    - ``hash_block_size`` is the granularity at which ``Request.block_hashes``
      is computed. Single group: equals scheduler block size. Multiple groups:
      ``cache_config.hash_block_size`` override if set, else the GCD of group
      block sizes; every group's block size must be divisible by it. Returns
      the scheduler block size (i.e. disables finer hashing) if block hashing
      is inactive or a mamba group's block size diverges from the cache
      block size (mamba_cache_mode != "align").
    """
    cache_config = vllm_config.cache_config
    dcp = vllm_config.parallel_config.decode_context_parallel_size
    pcp = vllm_config.parallel_config.prefill_context_parallel_size
    groups = kv_cache_config.kv_cache_groups

    if len(groups) <= 1:  # Single group: block_size * dcp * pcp
        bs = cache_config.block_size * dcp * pcp
        return bs, bs

    if dcp != 1 or pcp != 1:
        raise ValueError(
            "Hybrid KV cache groups with multiple block sizes do not "
            "support context parallelism (dcp_world_size/pcp_world_size > 1)."
        )

    group_block_sizes = [g.kv_cache_spec.block_size for g in groups]
    scheduler_block_size = math.lcm(*group_block_sizes)

    # Block hashes are only consumed by prefix caching and KV connectors
    # (P/D, offloading); when neither is active, keep hash_block_size equal
    # to the scheduler block size.
    connector_enabled = vllm_config.kv_transfer_config is not None
    if not (cache_config.enable_prefix_caching or connector_enabled):
        return scheduler_block_size, scheduler_block_size

    # Mamba groups with block_size != cache_config.block_size
    # (mamba_cache_mode != "align") break divisibility; back off to the
    # scheduler block size.
    if any(
        isinstance(g.kv_cache_spec, MambaSpec)
        and g.kv_cache_spec.block_size != cache_config.block_size
        for g in groups
    ):
        return scheduler_block_size, scheduler_block_size

    requested = cache_config.hash_block_size
    hash_block_size = (
        requested if requested is not None else math.gcd(*group_block_sizes)
    )
    if any(bs % hash_block_size != 0 for bs in group_block_sizes):
        raise ValueError(
            f"Invalid hash_block_size={hash_block_size}; all KV cache group "
            f"block sizes must be divisible by hash_block_size. "
            f"Got group block sizes={group_block_sizes}."
        )
    return scheduler_block_size, hash_block_size
```
**EN:** This function resolves configuration or runtime choices within the module. The docstring frames it as: Resolve (scheduler_block_size, hash_block_size). Key calls include `lcm`, `any`, `len`, `ValueError`, `gcd`, `isinstance`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会解析配置或运行期选择，其作用域位于the module。 关键调用包括 `lcm`, `any`, `len`, `ValueError`, `gcd`, `isinstance`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_check_enough_kv_cache_memory` function / `_check_enough_kv_cache_memory` 函数
```python
def _check_enough_kv_cache_memory(
    available_memory: int,
    get_needed_memory: Callable[[], int],
    max_model_len: int,
    estimate_max_model_len: Callable[[int], int],
):
    if available_memory <= 0:
        raise ValueError(
            "No available memory for the cache blocks. "
            "Try increasing `gpu_memory_utilization` when initializing the engine "
            "(this flag also controls CPU memory reservation on the CPU "
            "backend, despite its name). "
            "See https://docs.vllm.ai/en/latest/configuration/conserving_memory/ "
            "for more details."
        )

    needed_memory = get_needed_memory()

    if needed_memory > available_memory:
        estimated_max_len = estimate_max_model_len(available_memory)
        estimated_msg = ""
        if estimated_max_len > 0:
            estimated_msg = (
                "Based on the available memory, "
                f"the estimated maximum model length is {estimated_max_len}. "
            )

        raise ValueError(
            f"To serve at least one request with the model's max seq len "
            f"({max_model_len}), ({format_gib(needed_memory)} GiB KV "
            f"cache is needed, which is larger than the available KV cache "
            f"memory ({format_gib(available_memory)} GiB). {estimated_msg}"
            f"Try increasing `gpu_memory_utilization` (which also controls "
            f"CPU memory on the CPU backend) or decreasing `max_model_len` "
            f"when initializing the engine. "
            f"See https://docs.vllm.ai/en/latest/configuration/conserving_memory/ "
            f"for more details."
        )
```
**EN:** This function implements `_check_enough_kv_cache_memory` within the module. Key calls include `get_needed_memory`, `ValueError`, `estimate_max_model_len`, `format_gib`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_check_enough_kv_cache_memory`，其作用域位于the module。 关键调用包括 `get_needed_memory`, `ValueError`, `estimate_max_model_len`, `format_gib`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `check_enough_kv_cache_memory` function / `check_enough_kv_cache_memory` 函数
```python
def check_enough_kv_cache_memory(
    vllm_config: VllmConfig,
    kv_cache_spec: dict[str, KVCacheSpec],
    available_memory: int,
):
    """
    Checks whether `available_memory` is enough for the KV cache to hold at
    least one request with the model's max_model_len.

    Args:
        vllm_config: The global VllmConfig
        kv_cache_spec: The kv cache spec of each attention layer in the model
        available_memory: Memory available for KV cache in bytes.

    Raises:
        ValueError: If there is not enough memory available for the KV cache.
    """

    # No need to check for available memory if the kv_cache_spec is empty
    if kv_cache_spec:
        _check_enough_kv_cache_memory(
            available_memory,
            lambda: max_memory_usage_bytes(vllm_config, kv_cache_spec.values()),
            vllm_config.model_config.max_model_len,
            lambda am: estimate_max_model_len(vllm_config, kv_cache_spec, am),
        )
```
**EN:** This function validates assumptions or constraints within the module. The docstring frames it as: Checks whether `available_memory` is enough for the KV cache to hold at least one request with the model's max_model_len. Key calls include `_check_enough_kv_cache_memory`, `max_memory_usage_bytes`, `estimate_max_model_len`, `values`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `_check_enough_kv_cache_memory`, `max_memory_usage_bytes`, `estimate_max_model_len`, `values`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `create_kv_cache_group_specs` function / `create_kv_cache_group_specs` 函数
```python
def create_kv_cache_group_specs(
    kv_cache_spec: dict[str, KVCacheSpec], grouped_layer_names: list[list[str]]
) -> list[KVCacheGroupSpec]:
    """
    Create KVCacheGroupSpec object for each kv cache group layer.
    The layers in the same group should share the same
    KVCacheSpec.

    Args:
        kv_cache_spec:
            A mapping from each layer name to its corresponding KVCacheSpec.
        grouped_layer_names:
            A list of kv cache groups, where each element is a list of layer
            names that belong to the same group and should share the same
            KVCacheSpec.
    Returns:
        A list of KVCacheGroupSpec objects, one for each group.
    """
    kv_cache_groups = []
    for layer_names_one_group in grouped_layer_names:
        layer_specs = [
            kv_cache_spec[layer_name] for layer_name in layer_names_one_group
        ]
        merged_layer_spec = layer_specs[0].merge(layer_specs)
        kv_cache_groups.append(
            KVCacheGroupSpec(layer_names_one_group, merged_layer_spec)
        )
    return kv_cache_groups
```
**EN:** This function creates a new object or plan within the module. The docstring frames it as: Create KVCacheGroupSpec object for each kv cache group layer. Key calls include `merge`, `append`, `KVCacheGroupSpec`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会创建新的对象或计划，其作用域位于the module。 关键调用包括 `merge`, `append`, `KVCacheGroupSpec`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_get_kv_cache_groups_uniform_page_size` function / `_get_kv_cache_groups_uniform_page_size` 函数
```python
def _get_kv_cache_groups_uniform_page_size(
    kv_cache_spec: dict[str, KVCacheSpec],
) -> list[KVCacheGroupSpec]:
    """
    Generates the KV cache groups for hybrid models with multiple
    attention types but still with a uniform page size (physical memory per
    block per layer) for all layers.

    Detailed explanation about kv cache management of hybrid models:
    The layers in the models are repeated with some patterns, e.g., a model
    with 10 full attention layers and 20 sliding window attention layers can be
    regarded as repeating the pattern (1 * full, 2 * sw) 10 times.
    The KVCacheManager allocates different block tables for each of the 3 layers
    in the pattern, and repeats each of them 10 times to generate the
    block_table for the 30 layers in the model.
    Therefore, we can group the layers in the model into 3 kv_cache_groups, each
    of which contains 10 layers in the model.
    The KVCacheManager allocates the block_table for each group based on its
    kv_cache spec, and the model runner applies the block table to each layer
    in the group.
    For example:
    1. A model only uses full attention. The pattern is
    (num_hidden_layers * full), so there is only one group and the block table
    is shared by all layers. It is already handled by
    `_get_kv_cache_config_uniform_type`.
    2. A model with 10 full attention layers and 20 sliding window
    attention layers. There are 3 layers in the pattern (1 * full, 2 * sw), so
    there are 3 kv_cache_groups, each of which represents 10 layers.

    To simplify the implementation, we make the following assumptions:
    1. Physical memory per block: Must be the same across all KV cache groups.
    Breaking this assumption is non-trivial due to memory fragmentation concerns
    when allocating blocks of different sizes.
    2. Tokens per block (block_size): Currently, we directly use
    `CacheConfig.block_size` for all layers. It can be extended to vary by KV
    cache group, but within each KV cache group, all layers must share the same
    block size.
    3. Physical memory per token per layer: This property is decided by model
    config. Currently we only support models that have the same physical memory
    per token per layer for all layers. Can be relaxed with a simple extension,
    but still need to keep physical memory per block the same for all groups.
    4. Number of layers per group: Currently assumed the same for all layers.
    Can be relaxed with a simple extension, but still need to keep physical
    memory per block the same for all groups.
    5. Attention type within groups: All layers in a group must share the same
    attention type. One exception is that, when
    `--disable-hybrid-kv-cache-manager` is true, the single group for full
    attention layers may also include attention layers using sliding window or
    LLaMA 4 local attention. See `unify_hybrid_kv_cache_specs` for more details.
    6. Support for multiple attention types: The design for most components is
# ... omitted for brevity ...
        group_size = max_num_layers
    grouped_layers = []
    for layers in same_type_layers.values():
        num_padding_layers = group_size - len(layers) % group_size
        if num_padding_layers != group_size:
            logger.warning(
                "Add %d padding layers, may waste at most %.2f%% KV cache memory",  # noqa
                num_padding_layers,
                num_padding_layers / len(layers) * 100,
            )
        num_groups = cdiv(len(layers), group_size)
        # In PP case, say if we have
        # - stage 0: full.0, sw.0, sw.1
        # - stage 1: full.1, sw.2, sw.3
        # We should have 3 groups: (full.0, full.1), (sw.0, sw.2), (sw.1, sw.3)
        # It can't be (full.0, full.1), (sw.0, sw.1), (sw.2, sw.3) because
        # the 3 groups in stage 0 will be (full.0), (sw.0, sw.1), (empty group)
        # and it will be padded to (full.0, padding), (sw.0, sw.1),
        # (padding, padding) to ensure the number of layers in each group is
        # the same and will cause memory waste.
        # To avoid this, we assign layers[i::num_groups] to the i-th group
        # instead of layers[i * group_size: (i + 1) * group_size]
        for i in range(num_groups):
            grouped_layers.append(layers[i::num_groups])
    return create_kv_cache_group_specs(kv_cache_spec, grouped_layers)
```
**EN:** This function implements `_get_kv_cache_groups_uniform_page_size` within the module. The docstring frames it as: Generates the KV cache groups for hybrid models with multiple attention types but still with a uniform page size (physical memory per block per layer) for all layers. Key calls include `defaultdict`, `items`, `min`, `max`, `values`, `create_kv_cache_group_specs`. The control flow contains 2 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_get_kv_cache_groups_uniform_page_size`，其作用域位于the module。 关键调用包括 `defaultdict`, `items`, `min`, `max`, `values`, `create_kv_cache_group_specs`。 控制流包含 2 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `unify_hybrid_kv_cache_specs` function / `unify_hybrid_kv_cache_specs` 函数
```python
def unify_hybrid_kv_cache_specs(kv_cache_spec: dict[str, KVCacheSpec]):
    """
    This function tries to convert the KV cache specs to one type if the model
    is a hybrid model with multiple type of KV cache. It will convert all
    SlidingWindowSpec to FullAttentionSpec if both types are present.

    Args:
        kv_cache_spec: The kv cache spec of each attention layer in the model
    """

    if is_kv_cache_spec_uniform(
        kv_cache_spec
    ) or UniformTypeKVCacheSpecs.is_uniform_type(kv_cache_spec):
        return

    logger.warning(
        "Hybrid KV cache manager is disabled for this hybrid model, "
        "This means we do not enable any optimizations for saving KV cache "
        "memory (e.g., dropping the KV cache outside the sliding window). "
        "The compute of layers like sliding window is still saved."
    )

    has_full_attention = any(
        isinstance(spec, FullAttentionSpec) for spec in kv_cache_spec.values()
    )
    has_sliding_window = any(
        isinstance(spec, SlidingWindowSpec) for spec in kv_cache_spec.values()
    )
    has_chunked_local_attention = any(
        isinstance(spec, ChunkedLocalAttentionSpec) for spec in kv_cache_spec.values()
    )
    has_swa_mla = any(
        isinstance(spec, SlidingWindowMLASpec) for spec in kv_cache_spec.values()
    )

    uniform_block_size: int | None = None
    if has_swa_mla:
        # For DeepseekV4, block sizes can be different for different KV cache groups.
        # E.g., Full MLA: 256; SWA MLA: 64; C4 partial states: 4, C128 states: 8.
        assert has_full_attention
        any_full_spec = next(
            iter(
                spec
                for spec in kv_cache_spec.values()
                if isinstance(spec, FullAttentionSpec)
            )
        )
        uniform_block_size = any_full_spec.block_size

    if has_full_attention and (has_sliding_window or has_chunked_local_attention):
# ... omitted for brevity ...
                    head_size=spec.head_size,
                    head_size_v=spec.head_size_v,
                    dtype=spec.dtype,
                    kv_quant_mode=spec.kv_quant_mode,
                    sliding_window=spec.sliding_window,
                    page_size_padded=spec.page_size_padded,
                )
            elif isinstance(spec, ChunkedLocalAttentionSpec):
                kv_cache_spec[layer_name] = FullAttentionSpec(
                    block_size=spec.block_size,
                    num_kv_heads=spec.num_kv_heads,
                    head_size=spec.head_size,
                    dtype=spec.dtype,
                    attention_chunk_size=spec.attention_chunk_size,
                    page_size_padded=spec.page_size_padded,
                )

    if not (
        is_kv_cache_spec_uniform(kv_cache_spec)
        or UniformTypeKVCacheSpecs.is_uniform_type(kv_cache_spec)
    ):
        raise ValueError(
            "Hybrid KV cache manager is disabled but failed to "
            "convert the KV cache specs to one unified type."
        )
```
**EN:** This function implements `unify_hybrid_kv_cache_specs` within the module. The docstring frames it as: This function tries to convert the KV cache specs to one type if the model is a hybrid model with multiple type of KV cache. Key calls include `warning`, `any`, `is_kv_cache_spec_uniform`, `is_uniform_type`, `next`, `items`. The control flow contains 8 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `unify_hybrid_kv_cache_specs`，其作用域位于the module。 关键调用包括 `warning`, `any`, `is_kv_cache_spec_uniform`, `is_uniform_type`, `next`, `items`。 控制流包含 8 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `generate_scheduler_kv_cache_config` function / `generate_scheduler_kv_cache_config` 函数
```python
def generate_scheduler_kv_cache_config(
    kv_cache_configs: list[KVCacheConfig],
) -> KVCacheConfig:
    """
    Generate the KV cache configuration for the scheduler.
    """
    assert all(
        [cfg.num_blocks == kv_cache_configs[0].num_blocks for cfg in kv_cache_configs]
    )
    # All workers have the same kv_cache_config except layer names, so use
    # an arbitrary one to initialize the scheduler.
    cfg = copy.deepcopy(kv_cache_configs[0])
    for group in cfg.kv_cache_groups:
        if isinstance(group.kv_cache_spec, UniformTypeKVCacheSpecs):
            # All layers in the UniformTypeKVCacheSpecs have the same type,
            # so use an arbitrary one to initialize the scheduler.
            group.kv_cache_spec = next(
                iter(group.kv_cache_spec.kv_cache_specs.values())
            )
    return cfg
```
**EN:** This function implements `generate_scheduler_kv_cache_config` within the module. The docstring frames it as: Generate the KV cache configuration for the scheduler. Key calls include `all`, `deepcopy`, `isinstance`, `next`, `iter`, `values`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `generate_scheduler_kv_cache_config`，其作用域位于the module。 关键调用包括 `all`, `deepcopy`, `isinstance`, `next`, `iter`, `values`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `get_kv_cache_configs` function / `get_kv_cache_configs` 函数
```python
def get_kv_cache_configs(
    vllm_config: VllmConfig,
    kv_cache_specs: list[dict[str, KVCacheSpec]],
    available_memory: list[int],
) -> list[KVCacheConfig]:
    """
    Generates the KV cache configurations for a model.
    Since we use a shared centralized controller for all workers, we need the
    `kv_cache_config` to be consistent across all workers to make sure
    the KV cache allocation can be applied to all workers. However, different
    workers may have different memory available, and different type of layers
    (when pipeline parallel is enabled). To handle the difference between
    workers, the current implementation is:
    1. Merge the KV cache specs of all workers to get the KVCacheSpecs for
       the whole model.
    2. Generate the KV cache groups based on the layer ratio of the whole model.
       This also handles spec unification for hybrid models.
    3. Handle auto-fit max_model_len and memory checks using per-worker
       projected groups to account for PP sharding.
    4. Generate the KV cache configs for each worker based on the KV cache
       grouping strategy. (This is reasonable because the layer ratio of
       different PP stages are similar.)
    5. Change the num_blocks of each worker to the smallest among all workers
       and shrink tensor sizes proportionally to avoid allocating unused memory.

    Args:
        vllm_config: The global VllmConfig
        kv_cache_specs: List of dict[layer_name, KVCacheSpec] for each worker.
        available_memory: Memory available for KV cache in bytes for each
            worker.

    Returns:
        The generated KVCacheConfigs for each worker.
    """

    # Merge the KV cache specs of all workers. Different PP stages may have
    # different layer names, and different TP ranks of the same PP stage should
    # have the same KV cache spec.
    merged_kv_cache_specs: dict[str, KVCacheSpec] = {}
    for kv_cache_spec_one_worker in kv_cache_specs:
        for layer_name, layer_spec in kv_cache_spec_one_worker.items():
            if layer_name not in merged_kv_cache_specs:
                merged_kv_cache_specs[layer_name] = layer_spec
            else:
                assert merged_kv_cache_specs[layer_name] == layer_spec, (
                    "The KV cache specs for the same layer are different "
                    "across workers. This is not supported yet."
                )

    # Get global KV cache groups. This also handles spec unification for
# ... omitted for brevity ...
        kv_cache_configs.append(
            get_kv_cache_config_from_groups(
                vllm_config, projected_groups, available_memory_one_worker
            )
        )

    # Change the num_blocks of each rank to the smallest among all ranks.
    # We also need to shrink the tensor size proportionally to avoid
    # allocating unused memory.
    min_num_blocks = min(
        kv_cache_config.num_blocks for kv_cache_config in kv_cache_configs
    )
    for kv_cache_config in kv_cache_configs:
        num_blocks_old = kv_cache_config.num_blocks
        kv_cache_config.num_blocks = min_num_blocks

        # Shrink tensor size proportionally
        for tensor in kv_cache_config.kv_cache_tensors:
            assert tensor.size % num_blocks_old == 0
            tensor.size = tensor.size // num_blocks_old * min_num_blocks

        if len(kv_cache_config.kv_cache_groups) > 0:
            _report_kv_cache_config(vllm_config, kv_cache_config)

    return kv_cache_configs
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Generates the KV cache configurations for a model. Key calls include `get_kv_cache_groups`, `zip`, `min`, `items`, `_project_kv_cache_groups_to_worker`, `_auto_fit_max_model_len`. The control flow contains 6 branch(es) and 7 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `get_kv_cache_groups`, `zip`, `min`, `items`, `_project_kv_cache_groups_to_worker`, `_auto_fit_max_model_len`。 控制流包含 6 个分支和 7 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `BlockHashListWithBlockSize` class / `BlockHashListWithBlockSize` 类
```python
class BlockHashListWithBlockSize:
    """
    Convert block-hash granularity from `hash_block_size` to `target_block_size`.
    Used when KV cache groups have different block sizes: `hash_block_size`
    is the size used to compute the original `block_hashes`; `target_block_size`
    is the group's actual block size.

    Currently, only scaling up by an integer factor is supported (i.e.,
    `target_block_size` is a multiple of `hash_block_size`). Conversion is
    performed lazily on access for efficiency, by concatenating consecutive
    hashes at `hash_block_size` to form each hash at `target_block_size`.

    Example (`hash_block_size` = 16, `target_block_size` = 32):
    concatenating two 16-size hashes yields one 32-size hash:

    Block hashes with block_size 16:
    | Token Range | 0-15 | 16-31 | 32-47 | 48-63 |
    |-------------|------|-------|-------|-------|
    | Hash        | A    | B     | C     | D     |

    Block hashes with block_size 32:
    | Token Range | 0-31 | 32-63 |
    |-------------|------|-------|
    | Hash        | AB   | CD    |

    Args:
        block_hashes: Block hashes to convert, computed at `hash_block_size`.
        hash_block_size: Block size at which `block_hashes` were computed.
        target_block_size: Desired block size; must be a multiple of `hash_block_size`.
    """
```
**EN:** Introduces the `BlockHashListWithBlockSize` class. Core methods include `__init__`, `__len__`, `__getitem__`, `__getitem__`, `__getitem__`, `__iter__`. Docstring signal: Convert block-hash granularity from `hash_block_size` to `target_block_size`.
**CN:** 这里定义 `BlockHashListWithBlockSize` 类。核心方法包括 `__init__`, `__len__`, `__getitem__`, `__getitem__`, `__getitem__`, `__iter__`。

### `BlockHashListWithBlockSize.__init__` method / `BlockHashListWithBlockSize.__init__` 方法
```python
    def __init__(
        self,
        block_hashes: list[BlockHash],
        hash_block_size: int,
        target_block_size: int,
    ):
        self.block_hashes = block_hashes
        assert target_block_size % hash_block_size == 0
        self.scale_factor = target_block_size // hash_block_size
```
**EN:** This method initializes the object state within `BlockHashListWithBlockSize`. It touches state such as `block_hashes`, `scale_factor`.
**CN:** 该方法会初始化对象状态，其作用域位于`BlockHashListWithBlockSize`。 它会读写 `block_hashes`, `scale_factor` 等状态。

### `BlockHashListWithBlockSize.__len__` method / `BlockHashListWithBlockSize.__len__` 方法
```python
    def __len__(self) -> int:
        return len(self.block_hashes) // self.scale_factor
```
**EN:** This method implements `__len__` within `BlockHashListWithBlockSize`. Key calls include `len`.
**CN:** 该方法会实现 `__len__`，其作用域位于`BlockHashListWithBlockSize`。 关键调用包括 `len`。

### `BlockHashListWithBlockSize.__getitem__` method / `BlockHashListWithBlockSize.__getitem__` 方法
```python
    def __getitem__(self, idx):
        if isinstance(idx, int):
            return self._get_value_at(idx)

        if isinstance(idx, slice):
            start, stop, step = idx.indices(len(self))
            return [self._get_value_at(i) for i in range(start, stop, step)]

        raise TypeError(f"Invalid index type: {type(idx)!r}")
```
**EN:** This method implements `__getitem__` within `BlockHashListWithBlockSize`. Key calls include `isinstance`, `TypeError`, `_get_value_at`, `indices`, `len`, `range`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__getitem__`，其作用域位于`BlockHashListWithBlockSize`。 关键调用包括 `isinstance`, `TypeError`, `_get_value_at`, `indices`, `len`, `range`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BlockHashListWithBlockSize.__iter__` method / `BlockHashListWithBlockSize.__iter__` 方法
```python
    def __iter__(self) -> Iterator[BlockHash]:
        for i in range(len(self)):
            yield self._get_value_at(i)
```
**EN:** This method implements `__iter__` within `BlockHashListWithBlockSize`. Key calls include `range`, `len`, `_get_value_at`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__iter__`，其作用域位于`BlockHashListWithBlockSize`。 关键调用包括 `range`, `len`, `_get_value_at`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `BlockHashListWithBlockSize._get_value_at` method / `BlockHashListWithBlockSize._get_value_at` 方法
```python
    def _get_value_at(self, idx: int) -> BlockHash:
        base = idx * self.scale_factor
        end = base + self.scale_factor
        return BlockHash(b"".join(self.block_hashes[base:end]))
```
**EN:** This method implements `_get_value_at` within `BlockHashListWithBlockSize`. Key calls include `BlockHash`, `join`.
**CN:** 该方法会实现 `_get_value_at`，其作用域位于`BlockHashListWithBlockSize`。 关键调用包括 `BlockHash`, `join`。

### Module constants / 模块常量
```python
BlockHashList = list[BlockHash] | BlockHashListWithBlockSize
```
**EN:** Defines module-level constants or aliases such as `BlockHashList`, which are reused by later definitions.
**CN:** 定义 `BlockHashList` 等模块级常量或别名，供后续定义复用。

## Key Concepts / 关键概念
- `make_block_hash_with_group_id`: top-level helper or orchestration entry point. / `make_block_hash_with_group_id`：顶层辅助函数或编排入口。
- `get_block_hash`: top-level helper or orchestration entry point. / `get_block_hash`：顶层辅助函数或编排入口。
- `get_group_id`: top-level helper or orchestration entry point. / `get_group_id`：顶层辅助函数或编排入口。
- `maybe_convert_block_hash`: top-level helper or orchestration entry point. / `maybe_convert_block_hash`：顶层辅助函数或编排入口。
- `init_none_hash`: top-level helper or orchestration entry point. / `init_none_hash`：顶层辅助函数或编排入口。
- `KVCacheBlock`: central class or interface in this module. / `KVCacheBlock`：本模块中的核心类或接口。
- `FreeKVCacheBlockQueue`: central class or interface in this module. / `FreeKVCacheBlockQueue`：本模块中的核心类或接口。
- `need_extra_keys`: top-level helper or orchestration entry point. / `need_extra_keys`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `copy`, `hashlib`, `math`, `os`, `collections`, `dataclasses`, `functools`, `typing`
- Internal vLLM / 内部依赖: `vllm`, `vllm.config`, `vllm.logger`, `vllm.utils.hashing`, `vllm.utils.math_utils`, `vllm.utils.mem_utils`, `vllm.utils.torch_utils`, `vllm.v1.kv_cache_interface`, `vllm.v1.request`, `vllm.v1.utils`
