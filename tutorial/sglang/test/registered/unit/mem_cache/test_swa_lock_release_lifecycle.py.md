# test_swa_lock_release_lifecycle.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_swa_lock_release_lifecycle.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates swa lock release lifecycle behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 swa lock release lifecycle 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: supporting statements / 辅助语句
```python
"""Regression for SWA lock release lifecycle.

Hybrid-SWA early-release protocol: once a request's decode position passes
the sliding window, drop its prefill SWA lock without touching the full
lock, freeing SWA pages back to LRU.

Covers:
- SWARadixCache.dec_swa_lock_only (leaf tombstone + free, internal protected->evictable)
- SWARadixCache.dec_lock_ref(skip_swa=True)
- SWARadixCache.evict swa branch for leaf with full_lock_ref > 0
- SWARadixCache._delete_leaf skipping swa_evictable_size_ on tombstoned leaves
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 14-31: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.mem_cache.base_prefix_cache import (
    DecLockRefParams,
    EvictParams,
    InsertParams,
    MatchPrefixParams,
)
from sglang.srt.mem_cache.cache_init_params import CacheInitParams
from sglang.srt.mem_cache.memory_pool import ReqToTokenPool
from sglang.srt.mem_cache.radix_cache import RadixKey
from sglang.srt.mem_cache.swa_memory_pool import SWAKVPool, SWATokenToKVPoolAllocator
from sglang.srt.mem_cache.swa_radix_cache import SWARadixCache
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cache_init_params`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cache_init_params`。

### Lines 33-33: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=12, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 36-83: function build tree / 函数 build tree
```python
def _build_tree(
    *,
    sliding_window_size: int = 4,
    page_size: int = 1,
    kv_size: int = 128,
    kv_size_swa: int = 64,
):
    head_num, head_dim, num_layers, global_interval = 8, 128, 24, 4
    dtype = torch.bfloat16
    device = get_device()
    full_ids = list(range(0, num_layers, global_interval))
    swa_ids = [i for i in range(num_layers) if i not in set(full_ids)]

    pool = ReqToTokenPool(
        size=8, max_context_len=256, device=device, enable_memory_saver=False
    )
    kv_pool = SWAKVPool(
        size=kv_size,
        size_swa=kv_size_swa,
        page_size=page_size,
        dtype=dtype,
        head_num=head_num,
        head_dim=head_dim,
        swa_attention_layer_ids=swa_ids,
        full_attention_layer_ids=full_ids,
        enable_kvcache_transpose=False,
        device=device,
    )
    allocator = SWATokenToKVPoolAllocator(
        size=kv_size,
        size_swa=kv_size_swa,
        page_size=page_size,
        dtype=dtype,
        device=device,
        kvcache=kv_pool,
        need_sort=False,
    )
    tree = SWARadixCache(
        params=CacheInitParams(
            req_to_token_pool=pool,
            token_to_kv_pool_allocator=allocator,
            page_size=page_size,
            disable=False,
            is_eagle=False,
            sliding_window_size=sliding_window_size,
        ),
    )
    return tree, allocator, pool
```
**EN:** This block implements `_build_tree` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_tree`，承担模块行为中的一个聚焦逻辑片段。

### Lines 86-109: function swa alloc / 函数 swa alloc
```python
def _swa_alloc(allocator, need_size):
    """Allocate from SWA allocator for any page_size.

    SWATokenToKVPoolAllocator.alloc() asserts page_size == 1; for page_size > 1
    we drive the underlying paged allocators directly (mirrors the helper in
    test_swa_eviction_boundary.py). Required: need_size is a multiple of
    page_size when page_size > 1.
    """
    if allocator.page_size == 1:
        return allocator.alloc(need_size)

    assert need_size % allocator.page_size == 0, (
        f"page_size > 1 requires page-aligned alloc, got {need_size=} "
        f"with {allocator.page_size=}"
    )
    if need_size > allocator.full_attn_allocator.available_size():
        return None
    if need_size > allocator.swa_attn_allocator.available_size():
        return None
    full_indices = allocator.full_attn_allocator.alloc(need_size)
    swa_indices = allocator.swa_attn_allocator.alloc(need_size)
    assert full_indices is not None and swa_indices is not None
    allocator.full_to_swa_index_mapping[full_indices] = swa_indices
    return full_indices
```
**EN:** Allocate from SWA allocator for any page_size. This block implements `_swa_alloc` and captures one focused piece of the module's behavior.
**CN:** Allocate from SWA allocator for any page_size. 该代码块实现 `_swa_alloc`，承担模块行为中的一个聚焦逻辑片段。

### Lines 112-117: function insert chain / 函数 insert chain
```python
def _insert_chain(tree, allocator, token_ids):
    indices = _swa_alloc(allocator, len(token_ids))
    assert indices is not None
    tree.insert(InsertParams(key=RadixKey(token_ids), value=indices))
    match = tree.match_prefix(MatchPrefixParams(key=RadixKey(token_ids)))
    return match.last_device_node
```
**EN:** This block implements `_insert_chain` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_insert_chain`，承担模块行为中的一个聚焦逻辑片段。

### Lines 120-134: function release swa lock chain in place / 函数 release swa lock chain in place
```python
def _release_swa_lock_chain_in_place(tree, leaf, swa_uuid_for_lock):
    # Mirrors dec_swa_lock_only's non-tombstone arm (protected->evictable on
    # internal nodes) but skips the leaf-free + tombstone step, to construct
    # the post-revival state where SWA was already early-released yet the
    # leaf is back in swa_lru_list with full_lock_ref still > 0.
    node = leaf
    while node is not tree.root_node:
        if node.swa_lock_ref > 0:
            if node.swa_lock_ref == 1:
                tree.swa_protected_size_ -= len(node.value)
                tree.swa_evictable_size_ += len(node.value)
            node.swa_lock_ref -= 1
            if swa_uuid_for_lock and node.swa_uuid == swa_uuid_for_lock:
                break
        node = node.parent
```
**EN:** This block implements `_release_swa_lock_chain_in_place` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_release_swa_lock_chain_in_place`，承担模块行为中的一个聚焦逻辑片段。

### Lines 137-137: class TestSWALockReleaseLifecycle declaration / 类 TestSWALockReleaseLifecycle 声明
```python
class TestSWALockReleaseLifecycle(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 138-140: supporting statements / 辅助语句
```python
    """Each test pins one component of the early-release fix; method names
    are prefixed with the API surface they exercise so pytest output groups
    them naturally."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 142-173: test case dec swa lock only leaf tombstones and frees / 测试用例 dec swa lock only leaf tombstones and frees
```python
    def test_dec_swa_lock_only_leaf_tombstones_and_frees(self):
        tree, allocator, _ = _build_tree(sliding_window_size=4)
        leaf = _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6, 7, 8])
        self.assertEqual(len(leaf.value), 8)

        inc_res = tree.inc_lock_ref(leaf)
        swa_uuid = inc_res.swa_uuid_for_lock
        self.assertIsNotNone(swa_uuid)

        swa_avail_before = allocator.swa_available_size()
        full_avail_before = allocator.full_available_size()
        self.assertEqual(leaf.swa_lock_ref, 1)
        self.assertEqual(leaf.full_lock_ref, 1)
        self.assertFalse(leaf.swa_tombstone)
        self.assertTrue(tree.swa_lru_list.in_list(leaf))

        tree.dec_swa_lock_only(leaf, swa_uuid_for_lock=swa_uuid)

        self.assertTrue(leaf.swa_tombstone)
        self.assertFalse(tree.swa_lru_list.in_list(leaf))
        self.assertEqual(leaf.swa_lock_ref, 0)
        self.assertEqual(
            allocator.swa_available_size(), swa_avail_before + len(leaf.value)
        )
        self.assertEqual(leaf.full_lock_ref, 1)
        self.assertEqual(allocator.full_available_size(), full_avail_before)

        # sanity_check forbids live locks; release the full half before checking.
        tree.dec_lock_ref(
            leaf, DecLockRefParams(swa_uuid_for_lock=swa_uuid), skip_swa=True
        )
        tree.sanity_check()
```
**EN:** This test exercises `test_dec_swa_lock_only_leaf_tombstones_and_frees` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_swa_lock_only_leaf_tombstones_and_frees`。

### Lines 175-212: test case dec swa lock only internal no tombstone no free / 测试用例 dec swa lock only internal no tombstone no free
```python
    def test_dec_swa_lock_only_internal_no_tombstone_no_free(self):
        # Two siblings force an internal node at the shared prefix.
        tree, allocator, _ = _build_tree(sliding_window_size=4)
        leaf_a = _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6, 7, 8])
        _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6, 7, 9])

        # Post-split: leaf_a now carries [8] only, parent holds the shared 7.
        self.assertEqual(len(leaf_a.value), 1)
        internal = leaf_a.parent
        self.assertGreater(len(internal.children), 1)
        self.assertEqual(len(internal.value), 7)

        inc_res = tree.inc_lock_ref(leaf_a)
        swa_uuid = inc_res.swa_uuid_for_lock
        # window=4, value 1 (leaf) + 7 (internal): swa lock chain ends at internal.
        self.assertEqual(swa_uuid, internal.swa_uuid)

        swa_protected_before = tree.swa_protected_size_
        swa_evictable_before = tree.swa_evictable_size_
        swa_avail_before = allocator.swa_available_size()

        tree.dec_swa_lock_only(leaf_a, swa_uuid_for_lock=swa_uuid)

        self.assertFalse(internal.swa_tombstone)
        self.assertTrue(tree.swa_lru_list.in_list(internal))
        self.assertEqual(internal.swa_lock_ref, 0)
        self.assertEqual(
            tree.swa_protected_size_, swa_protected_before - (len(leaf_a.value) + 7)
        )
        self.assertEqual(tree.swa_evictable_size_, swa_evictable_before + 7)
        self.assertEqual(
            allocator.swa_available_size(), swa_avail_before + len(leaf_a.value)
        )

        tree.dec_lock_ref(
            leaf_a, DecLockRefParams(swa_uuid_for_lock=swa_uuid), skip_swa=True
        )
        tree.sanity_check()
```
**EN:** This test exercises `test_dec_swa_lock_only_internal_no_tombstone_no_free` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_swa_lock_only_internal_no_tombstone_no_free`。

### Lines 214-236: test case dec lock ref skip swa true drops full only / 测试用例 dec lock ref skip swa true drops full only
```python
    def test_dec_lock_ref_skip_swa_true_drops_full_only(self):
        tree, allocator, _ = _build_tree(sliding_window_size=4)
        leaf = _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6, 7, 8])

        inc_res = tree.inc_lock_ref(leaf)
        swa_uuid = inc_res.swa_uuid_for_lock

        tree.dec_swa_lock_only(leaf, swa_uuid_for_lock=swa_uuid)
        self.assertTrue(leaf.swa_tombstone)
        self.assertEqual(leaf.full_lock_ref, 1)

        swa_avail_after_release = allocator.swa_available_size()
        swa_protected_after_release = tree.swa_protected_size_

        # Without skip_swa, dec_lock_ref would assert on the swa_tombstone leaf.
        tree.dec_lock_ref(
            leaf, DecLockRefParams(swa_uuid_for_lock=swa_uuid), skip_swa=True
        )

        self.assertEqual(leaf.full_lock_ref, 0)
        self.assertEqual(allocator.swa_available_size(), swa_avail_after_release)
        self.assertEqual(tree.swa_protected_size_, swa_protected_after_release)
        tree.sanity_check()
```
**EN:** This test exercises `test_dec_lock_ref_skip_swa_true_drops_full_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_lock_ref_skip_swa_true_drops_full_only`。

### Lines 238-258: test case dec lock ref skip swa false drops both / 测试用例 dec lock ref skip swa false drops both
```python
    def test_dec_lock_ref_skip_swa_false_drops_both(self):
        # Default skip_swa=False must keep legacy behavior intact.
        tree, allocator, _ = _build_tree(sliding_window_size=4)
        leaf = _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6, 7, 8])

        inc_res = tree.inc_lock_ref(leaf)
        swa_uuid = inc_res.swa_uuid_for_lock

        full_avail_before = allocator.full_available_size()
        swa_avail_before = allocator.swa_available_size()

        tree.dec_lock_ref(leaf, DecLockRefParams(swa_uuid_for_lock=swa_uuid))

        self.assertEqual(leaf.full_lock_ref, 0)
        self.assertEqual(leaf.swa_lock_ref, 0)
        self.assertEqual(tree.full_protected_size_, 0)
        self.assertEqual(tree.swa_protected_size_, 0)
        # dec_lock_ref releases locks but doesn't free; eviction does.
        self.assertEqual(allocator.full_available_size(), full_avail_before)
        self.assertEqual(allocator.swa_available_size(), swa_avail_before)
        tree.sanity_check()
```
**EN:** This test exercises `test_dec_lock_ref_skip_swa_false_drops_both` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_lock_ref_skip_swa_false_drops_both`。

### Lines 260-298: test case evict swa leaf with full lock tombstones in place / 测试用例 evict swa leaf with full lock tombstones in place
```python
    def test_evict_swa_leaf_with_full_lock_tombstones_in_place(self):
        # Large window so inc_lock_ref locks the entire SWA chain.
        tree, allocator, _ = _build_tree(sliding_window_size=64)
        leaf = _insert_chain(tree, allocator, [1, 2, 3, 4])
        self.assertEqual(len(leaf.value), 4)

        inc_res = tree.inc_lock_ref(leaf)
        _release_swa_lock_chain_in_place(tree, leaf, inc_res.swa_uuid_for_lock)

        self.assertEqual(leaf.full_lock_ref, 1)
        self.assertEqual(leaf.swa_lock_ref, 0)
        self.assertFalse(leaf.swa_tombstone)
        self.assertTrue(tree.swa_lru_list.in_list(leaf))

        swa_avail_before = allocator.swa_available_size()
        swa_evictable_before = tree.swa_evictable_size_

        # num_tokens=0 skips the full eviction loop; swa loop hits the new branch.
        evict_res = tree.evict(EvictParams(num_tokens=0, swa_num_tokens=4))

        self.assertGreaterEqual(evict_res.swa_num_tokens_evicted, 4)
        self.assertTrue(leaf.swa_tombstone)
        self.assertFalse(tree.swa_lru_list.in_list(leaf))
        self.assertEqual(leaf.full_lock_ref, 1)
        self.assertEqual(
            allocator.swa_available_size(), swa_avail_before + len(leaf.value)
        )
        # Full lock prevents _delete_leaf, so the node stays attached.
        self.assertIs(leaf.parent.children[leaf.key.child_key(tree.page_size)], leaf)
        self.assertEqual(
            tree.swa_evictable_size_, swa_evictable_before - len(leaf.value)
        )

        tree.dec_lock_ref(
            leaf,
            DecLockRefParams(swa_uuid_for_lock=inc_res.swa_uuid_for_lock),
            skip_swa=True,
        )
        tree.sanity_check()
```
**EN:** This test exercises `test_evict_swa_leaf_with_full_lock_tombstones_in_place` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_evict_swa_leaf_with_full_lock_tombstones_in_place`。

### Lines 300-315: test case delete leaf skips swa size on tombstone / 测试用例 delete leaf skips swa size on tombstone
```python
    def test_delete_leaf_skips_swa_size_on_tombstone(self):
        # Tombstone removes the count once; _delete_leaf must not subtract again.
        tree, allocator, _ = _build_tree(sliding_window_size=4)
        leaf = _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6, 7, 8])

        inc_res = tree.inc_lock_ref(leaf)
        swa_uuid = inc_res.swa_uuid_for_lock

        tree.dec_swa_lock_only(leaf, swa_uuid_for_lock=swa_uuid)
        self.assertTrue(leaf.swa_tombstone)

        swa_evictable_before_delete = tree.swa_evictable_size_
        tree.full_lru_list.remove_node(leaf)
        tree._delete_leaf(leaf)

        self.assertEqual(tree.swa_evictable_size_, swa_evictable_before_delete)
```
**EN:** This test exercises `test_delete_leaf_skips_swa_size_on_tombstone` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_delete_leaf_skips_swa_size_on_tombstone`。

### Lines 317-372: test case dec swa lock only leaf page size variants / 测试用例 dec swa lock only leaf page size variants
```python
    def test_dec_swa_lock_only_leaf_page_size_variants(self):
        """Single-leaf tombstone+free across all (page_size, window) regimes.

        Sweep covers:
          - window multiple of page_size  (page_size=2, window=4)
          - page_size > window            (page_size=8, window=4)
          - window not multiple of page   (page_size=4, window=6)

        With page_size > 1, _swa_alloc routes through the paged allocators;
        free_swa(leaf.value) must release exactly len(leaf.value) tokens
        (page-aligned) regardless of how page_size relates to the window.
        """
        for page_size, window in [(2, 4), (8, 4), (4, 6)]:
            with self.subTest(page_size=page_size, window=window):
                tree, allocator, _ = _build_tree(
                    sliding_window_size=window,
                    page_size=page_size,
                    kv_size=max(128, 32 * page_size),
                    kv_size_swa=max(64, 16 * page_size),
                )
                n_tokens = max(window, 2 * page_size)
                n_tokens = (n_tokens + page_size - 1) // page_size * page_size
                leaf = _insert_chain(tree, allocator, list(range(1, n_tokens + 1)))
                self.assertEqual(len(leaf.value), n_tokens)
                self.assertEqual(len(leaf.value) % page_size, 0)

                inc_res = tree.inc_lock_ref(leaf)
                swa_uuid = inc_res.swa_uuid_for_lock
                self.assertIsNotNone(
                    swa_uuid,
                    f"inc_lock_ref must reach the window with leaf.value="
                    f"{len(leaf.value)} >= window={window}",
                )

                swa_avail_before = allocator.swa_available_size()
                full_avail_before = allocator.full_available_size()

                tree.dec_swa_lock_only(leaf, swa_uuid_for_lock=swa_uuid)

                self.assertTrue(leaf.swa_tombstone)
                self.assertFalse(tree.swa_lru_list.in_list(leaf))
                self.assertEqual(leaf.swa_lock_ref, 0)
                self.assertEqual(
                    allocator.swa_available_size(),
                    swa_avail_before + len(leaf.value),
                    "free_swa must release the leaf's full page-aligned slot count",
                )
                self.assertEqual(leaf.full_lock_ref, 1)
                self.assertEqual(allocator.full_available_size(), full_avail_before)

                tree.dec_lock_ref(
                    leaf,
                    DecLockRefParams(swa_uuid_for_lock=swa_uuid),
                    skip_swa=True,
                )
                tree.sanity_check()
```
**EN:** Single-leaf tombstone+free across all (page_size, window) regimes. This test exercises `test_dec_swa_lock_only_leaf_page_size_variants` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Single-leaf tombstone+free across all (page_size, window) regimes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_swa_lock_only_leaf_page_size_variants`。

### Lines 374-431: test case dec swa lock only internal page size gt 1 / 测试用例 dec swa lock only internal page size gt 1
```python
    def test_dec_swa_lock_only_internal_page_size_gt_1(self):
        """Internal-node chain release with page_size > 1.

        Two siblings sharing a page-aligned prefix force a radix split on a
        page boundary. The swa lock chain therefore spans leaf -> internal,
        and dec_swa_lock_only must:
          - tombstone the leaf and free len(leaf.value) SWA tokens
          - flip the internal node from protected -> evictable (no free,
            no tombstone)
        """
        page_size, window = 2, 6
        tree, allocator, _ = _build_tree(
            sliding_window_size=window, page_size=page_size
        )
        # Shared prefix len 4 (2 pages); divergent suffix len 2 (1 page each).
        leaf_a = _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6])
        _insert_chain(tree, allocator, [1, 2, 3, 4, 7, 8])

        self.assertEqual(len(leaf_a.value), 2)
        internal = leaf_a.parent
        self.assertGreater(len(internal.children), 1)
        self.assertEqual(len(internal.value), 4)

        inc_res = tree.inc_lock_ref(leaf_a)
        swa_uuid = inc_res.swa_uuid_for_lock
        # leaf_a (2) + internal (4) = 6 >= window=6, so uuid stops at internal.
        self.assertEqual(swa_uuid, internal.swa_uuid)

        swa_protected_before = tree.swa_protected_size_
        swa_evictable_before = tree.swa_evictable_size_
        swa_avail_before = allocator.swa_available_size()

        tree.dec_swa_lock_only(leaf_a, swa_uuid_for_lock=swa_uuid)

        # Leaf side: tombstoned and pages freed.
        self.assertTrue(leaf_a.swa_tombstone)
        self.assertFalse(tree.swa_lru_list.in_list(leaf_a))
        self.assertEqual(
            allocator.swa_available_size(),
            swa_avail_before + len(leaf_a.value),
        )
        # Internal side: protected -> evictable, still in lru, no free.
        self.assertFalse(internal.swa_tombstone)
        self.assertTrue(tree.swa_lru_list.in_list(internal))
        self.assertEqual(internal.swa_lock_ref, 0)
        self.assertEqual(
            tree.swa_protected_size_,
            swa_protected_before - (len(leaf_a.value) + len(internal.value)),
        )
        self.assertEqual(
            tree.swa_evictable_size_,
            swa_evictable_before + len(internal.value),
        )

        tree.dec_lock_ref(
            leaf_a, DecLockRefParams(swa_uuid_for_lock=swa_uuid), skip_swa=True
        )
        tree.sanity_check()
```
**EN:** Internal-node chain release with page_size > 1. This test exercises `test_dec_swa_lock_only_internal_page_size_gt_1` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Internal-node chain release with page_size > 1. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_swa_lock_only_internal_page_size_gt_1`。

### Lines 433-462: test case full lifecycle inc dec swa dec lock balances / 测试用例 full lifecycle inc dec swa dec lock balances
```python
    def test_full_lifecycle_inc_dec_swa_dec_lock_balances(self):
        tree, allocator, _ = _build_tree(sliding_window_size=4)
        leaf = _insert_chain(tree, allocator, [1, 2, 3, 4, 5, 6, 7, 8])

        full_protected0 = tree.full_protected_size_
        swa_protected0 = tree.swa_protected_size_
        full_avail0 = allocator.full_available_size()
        swa_avail0 = allocator.swa_available_size()

        inc_res = tree.inc_lock_ref(leaf)
        swa_uuid = inc_res.swa_uuid_for_lock

        self.assertGreater(tree.full_protected_size_, full_protected0)
        self.assertGreater(tree.swa_protected_size_, swa_protected0)

        tree.dec_swa_lock_only(leaf, swa_uuid_for_lock=swa_uuid)

        self.assertEqual(tree.swa_protected_size_, swa_protected0)
        self.assertGreater(tree.full_protected_size_, full_protected0)

        tree.dec_lock_ref(
            leaf, DecLockRefParams(swa_uuid_for_lock=swa_uuid), skip_swa=True
        )

        self.assertEqual(tree.full_protected_size_, full_protected0)
        self.assertEqual(tree.swa_protected_size_, swa_protected0)
        self.assertEqual(allocator.full_available_size(), full_avail0)
        self.assertEqual(allocator.swa_available_size(), swa_avail0 + len(leaf.value))

        tree.sanity_check()
```
**EN:** This test exercises `test_full_lifecycle_inc_dec_swa_dec_lock_balances` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_full_lifecycle_inc_dec_swa_dec_lock_balances`。

### Lines 465-466: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_build_tree`: This block implements `_build_tree` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_tree`，承担模块行为中的一个聚焦逻辑片段。
- `_swa_alloc`: Allocate from SWA allocator for any page_size. / 该代码块实现 `_swa_alloc`，承担模块行为中的一个聚焦逻辑片段。
- `_insert_chain`: This block implements `_insert_chain` and captures one focused piece of the module's behavior. / 该代码块实现 `_insert_chain`，承担模块行为中的一个聚焦逻辑片段。
- `_release_swa_lock_chain_in_place`: This block implements `_release_swa_lock_chain_in_place` and captures one focused piece of the module's behavior. / 该代码块实现 `_release_swa_lock_chain_in_place`，承担模块行为中的一个聚焦逻辑片段。
- `TestSWALockReleaseLifecycle`: Each test pins one component of the early-release fix; method names are prefixed with the API surface they exercise so pytest output groups them naturally. / 用于组织相关测试、夹具或辅助方法。
- `TestSWALockReleaseLifecycle.test_dec_swa_lock_only_leaf_tombstones_and_frees`: This test exercises `test_dec_swa_lock_only_leaf_tombstones_and_frees` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_swa_lock_only_leaf_tombstones_and_frees`。
- `TestSWALockReleaseLifecycle.test_dec_swa_lock_only_internal_no_tombstone_no_free`: This test exercises `test_dec_swa_lock_only_internal_no_tombstone_no_free` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_swa_lock_only_internal_no_tombstone_no_free`。
- `TestSWALockReleaseLifecycle.test_dec_lock_ref_skip_swa_true_drops_full_only`: This test exercises `test_dec_lock_ref_skip_swa_true_drops_full_only` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_lock_ref_skip_swa_true_drops_full_only`。
- `TestSWALockReleaseLifecycle.test_dec_lock_ref_skip_swa_false_drops_both`: This test exercises `test_dec_lock_ref_skip_swa_false_drops_both` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_lock_ref_skip_swa_false_drops_both`。
- `TestSWALockReleaseLifecycle.test_evict_swa_leaf_with_full_lock_tombstones_in_place`: This test exercises `test_evict_swa_leaf_with_full_lock_tombstones_in_place` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_evict_swa_leaf_with_full_lock_tombstones_in_place`。
- `TestSWALockReleaseLifecycle.test_delete_leaf_skips_swa_size_on_tombstone`: This test exercises `test_delete_leaf_skips_swa_size_on_tombstone` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_delete_leaf_skips_swa_size_on_tombstone`。
- `TestSWALockReleaseLifecycle.test_dec_swa_lock_only_leaf_page_size_variants`: Single-leaf tombstone+free across all (page_size, window) regimes. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dec_swa_lock_only_leaf_page_size_variants`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.mem_cache.swa_memory_pool`, `sglang.srt.mem_cache.swa_radix_cache`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 466
