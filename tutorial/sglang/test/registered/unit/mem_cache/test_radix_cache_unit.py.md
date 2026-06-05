# test_radix_cache_unit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_radix_cache_unit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates radix cache unit behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 radix cache unit 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: supporting statements / 辅助语句
```python
"""
Unit tests for the RadixCache implementation.

This module tests the core functionality of RadixCache, RadixKey, and TreeNode
following SGLang testing patterns.

Test Coverage:
- RadixKey: token ID management, slicing, iteration, representation
- TreeNode: node properties, reference counting, hash values
- RadixCache: insert/match operations, eviction, page alignment, error handling
- Cache events and request handling
- Boundary conditions with parameterized testing

Usage:
    python test_radix_cache_unit.py
    python -m pytest test_radix_cache_unit.py -v
    python -m pytest test_radix_cache_unit.py::TestRadixCache::test_insert_basic
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 20-21: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.mem_cache.common import available_and_evictable_str
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.mem_cache.common`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.mem_cache.common`, `sglang.test.ci.ci_register`。

### Lines 22-23: supporting source context / 辅助源码上下文
```python

# CPU-based unit test, runs quickly on any GPU runner
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 24-25: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=15, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=5, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-41: module imports and dependencies / 模块导入与依赖
```python
import random
import time
import unittest
import unittest.mock

import torch

from sglang.srt.disaggregation.kv_events import BlockRemoved, BlockStored
from sglang.srt.mem_cache.base_prefix_cache import (
    EvictParams,
    EvictResult,
    InsertParams,
    MatchPrefixParams,
)
from sglang.srt.mem_cache.radix_cache import RadixCache, RadixKey, TreeNode
```
**EN:** This block imports the modules needed by the rest of the file, including `random`, `time`, `unittest`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `random`, `time`, `unittest`, `unittest.mock`。

### Lines 42-43: supporting source context / 辅助源码上下文
```python

# Test constants
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 44-44: module-level constants and configuration / 模块级常量与配置
```python
DEFAULT_PAGE_SIZE = 4
```
**EN:** This block defines shared names such as `DEFAULT_PAGE_SIZE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `DEFAULT_PAGE_SIZE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 47-47: class TestRadixKey declaration / 类 TestRadixKey 声明
```python
class TestRadixKey(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 48-48: supporting statements / 辅助语句
```python
    """Test cases for RadixKey class."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 50-55: test case init basic / 测试用例 init basic
```python
    def test_init_basic(self):
        """Test basic initialization of RadixKey."""
        token_ids = [1, 2, 3, 4]
        key = RadixKey(token_ids)
        self.assertEqual(key.token_ids, token_ids)
        self.assertIsNone(key.extra_key)
```
**EN:** Test basic initialization of RadixKey. This test exercises `test_init_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic initialization of RadixKey. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_basic`。

### Lines 57-63: test case init with extra key / 测试用例 init with extra key
```python
    def test_init_with_extra_key(self):
        """Test initialization with extra_key."""
        token_ids = [1, 2, 3]
        extra_key = "test_key"
        key = RadixKey(token_ids, extra_key)
        self.assertEqual(key.token_ids, token_ids)
        self.assertEqual(key.extra_key, extra_key)
```
**EN:** Test initialization with extra_key. This test exercises `test_init_with_extra_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test initialization with extra_key. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_with_extra_key`。

### Lines 65-71: test case len / 测试用例 len
```python
    def test_len(self):
        """Test __len__ method."""
        key = RadixKey([1, 2, 3])
        self.assertEqual(len(key), 3)

        empty_key = RadixKey([])
        self.assertEqual(len(empty_key), 0)
```
**EN:** Test __len__ method. This test exercises `test_len` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __len__ method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_len`。

### Lines 73-77: test case iter / 测试用例 iter
```python
    def test_iter(self):
        """Test __iter__ method."""
        token_ids = [1, 2, 3, 4]
        key = RadixKey(token_ids)
        self.assertEqual(list(key), token_ids)
```
**EN:** Test __iter__ method. This test exercises `test_iter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __iter__ method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_iter`。

### Lines 79-91: test case len and iter / 测试用例 len and iter
```python
    def test_len_and_iter(self):
        """Test __len__ and __iter__ methods."""
        test_cases = [
            ([1, 2, 3], 3),
            ([], 0),
            ([42], 1),
        ]

        for tokens, expected in test_cases:
            with self.subTest(tokens=tokens):
                key = RadixKey(tokens)
                self.assertEqual(len(key), expected)
                self.assertEqual(list(key), tokens)
```
**EN:** Test __len__ and __iter__ methods. This test exercises `test_len_and_iter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __len__ and __iter__ methods. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_len_and_iter`。

### Lines 93-106: test case getitem int / 测试用例 getitem int
```python
    def test_getitem_int(self):
        """Test __getitem__ with int index."""
        test_cases = [
            ([10, 20, 30], 0, [10]),
            ([10, 20, 30], -1, [30]),
            ([10, 20, 30], 2, [30]),
        ]

        for tokens, index, expected in test_cases:
            with self.subTest(tokens=tokens, index=index):
                key = RadixKey(tokens)
                result = key[index]
                self.assertIsInstance(result, RadixKey)
                self.assertEqual(result.token_ids, expected)
```
**EN:** Test __getitem__ with int index. This test exercises `test_getitem_int` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __getitem__ with int index. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getitem_int`。

### Lines 108-120: test case getitem slice / 测试用例 getitem slice
```python
    def test_getitem_slice(self):
        """Test __getitem__ with slice and edge cases."""
        key = RadixKey([1, 2, 3, 4, 5], "extra")

        # Basic slice
        sliced = key[1:4]
        self.assertIsInstance(sliced, RadixKey)
        self.assertEqual(sliced.token_ids, [2, 3, 4])
        self.assertEqual(sliced.extra_key, "extra")

        # Edge cases
        self.assertEqual(key[2:2].token_ids, [])  # Empty slice
        self.assertEqual(key[:].token_ids, [1, 2, 3, 4, 5])  # Full slice
```
**EN:** Test __getitem__ with slice and edge cases. This test exercises `test_getitem_slice` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __getitem__ with slice and edge cases. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getitem_slice`。

### Lines 122-126: test case getitem invalid index / 测试用例 getitem invalid index
```python
    def test_getitem_invalid_index(self):
        """Test __getitem__ with invalid indices."""
        key = RadixKey([1, 2, 3])
        with self.assertRaises(IndexError):
            _ = key[10]  # Out of bounds
```
**EN:** Test __getitem__ with invalid indices. This test exercises `test_getitem_invalid_index` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __getitem__ with invalid indices. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getitem_invalid_index`。

### Lines 128-134: test case repr / 测试用例 repr
```python
    def test_repr(self):
        """Test __repr__ method."""
        key = RadixKey([1, 2, 3], "test")
        repr_str = repr(key)
        self.assertIn("RadixKey", repr_str)
        self.assertIn("extra_key='test'", repr_str)
        self.assertIn("[1, 2, 3]", repr_str)
```
**EN:** Test __repr__ method. This test exercises `test_repr` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __repr__ method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repr`。

### Lines 136-141: test case repr long token ids / 测试用例 repr long token ids
```python
    def test_repr_long_token_ids(self):
        """Test __repr__ with long token_ids."""
        long_tokens = list(range(15))
        key = RadixKey(long_tokens)
        repr_str = repr(key)
        self.assertIn("...", repr_str)  # Should be truncated
```
**EN:** Test __repr__ with long token_ids. This test exercises `test_repr_long_token_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test __repr__ with long token_ids. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repr_long_token_ids`。

### Lines 144-144: class TestTreeNode declaration / 类 TestTreeNode 声明
```python
class TestTreeNode(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 145-145: supporting statements / 辅助语句
```python
    """Test cases for TreeNode class."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 147-149: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Reset the counter before each test."""
        TreeNode.counter = 0
```
**EN:** Reset the counter before each test. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Reset the counter before each test. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 151-163: test case init basic / 测试用例 init basic
```python
    def test_init_basic(self):
        """Test basic initialization of TreeNode."""
        node = TreeNode()
        self.assertEqual(node.id, 0)
        self.assertEqual(len(node.children), 0)
        self.assertIsNone(node.parent)
        self.assertIsNone(node.key)
        self.assertIsNone(node.value)
        self.assertEqual(node.lock_ref, 0)
        self.assertEqual(node.hit_count, 0)
        self.assertEqual(node.host_ref_counter, 0)
        self.assertIsNone(node.host_value)
        self.assertIsNone(node.hash_value)
```
**EN:** Test basic initialization of TreeNode. This test exercises `test_init_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic initialization of TreeNode. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_basic`。

### Lines 165-170: test case init with id / 测试用例 init with id
```python
    def test_init_with_id(self):
        """Test initialization with custom ID."""
        node = TreeNode(id=42)
        self.assertEqual(node.id, 42)
        node2 = TreeNode()
        self.assertEqual(node2.id, 1)  # Counter was incremented
```
**EN:** Test initialization with custom ID. This test exercises `test_init_with_id` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test initialization with custom ID. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_with_id`。

### Lines 172-177: test case counter increment / 测试用例 counter increment
```python
    def test_counter_increment(self):
        """Test that counter increments properly."""
        node1 = TreeNode()
        node2 = TreeNode()
        self.assertEqual(node1.id, 0)
        self.assertEqual(node2.id, 1)
```
**EN:** Test that counter increments properly. This test exercises `test_counter_increment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that counter increments properly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_counter_increment`。

### Lines 179-203: test case evicted backuped properties / 测试用例 evicted backuped properties
```python
    def test_evicted_backuped_properties(self):
        """Test evicted and backuped properties."""
        test_cases = [
            (False, False, True, False),
            (True, False, False, False),
            (True, True, False, True),
            (False, True, True, True),
        ]

        for (
            has_value,
            has_host_value,
            expected_evicted,
            expected_backuped,
        ) in test_cases:
            with self.subTest(has_value=has_value, has_host_value=has_host_value):
                node = TreeNode()

                if has_value:
                    node.value = torch.tensor([1, 2, 3])
                if has_host_value:
                    node.host_value = torch.tensor([4, 5, 6])

                self.assertEqual(node.evicted, expected_evicted)
                self.assertEqual(node.backuped, expected_backuped)
```
**EN:** Test evicted and backuped properties. This test exercises `test_evicted_backuped_properties` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test evicted and backuped properties. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_evicted_backuped_properties`。

### Lines 205-218: test case protect release host / 测试用例 protect release host
```python
    def test_protect_release_host(self):
        """Test protect_host and release_host methods."""
        node = TreeNode()
        self.assertEqual(node.host_ref_counter, 0)

        node.protect_host()
        self.assertEqual(node.host_ref_counter, 1)

        node.release_host()
        self.assertEqual(node.host_ref_counter, 0)

        # Test error case
        with self.assertRaises(RuntimeError):
            node.release_host()
```
**EN:** Test protect_host and release_host methods. This test exercises `test_protect_release_host` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test protect_host and release_host methods. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_protect_release_host`。

### Lines 220-226: test case get last hash value / 测试用例 get last hash value
```python
    def test_get_last_hash_value(self):
        """Test get_last_hash_value method."""
        node = TreeNode()
        self.assertIsNone(node.get_last_hash_value())

        node.hash_value = ["hash1", "hash2", "hash3"]
        self.assertEqual(node.get_last_hash_value(), "hash3")
```
**EN:** Test get_last_hash_value method. This test exercises `test_get_last_hash_value` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test get_last_hash_value method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_last_hash_value`。

### Lines 228-235: test case lt comparison / 测试用例 lt comparison
```python
    def test_lt_comparison(self):
        """Test less than comparison based on last_access_time."""
        node1 = TreeNode()
        time.sleep(0.001)  # Small delay to ensure different timestamps
        node2 = TreeNode()

        self.assertTrue(node1 < node2)
        self.assertFalse(node2 < node1)
```
**EN:** Test less than comparison based on last_access_time. This test exercises `test_lt_comparison` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test less than comparison based on last_access_time. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lt_comparison`。

### Lines 238-238: class TestRadixCache declaration / 类 TestRadixCache 声明
```python
class TestRadixCache(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 239-239: supporting statements / 辅助语句
```python
    """Test cases for RadixCache class."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 241-243: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test fixtures."""
        TreeNode.counter = 0
```
**EN:** Set up test fixtures. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test fixtures. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 245-268: test case init variations / 测试用例 init variations
```python
    def test_init_variations(self):
        """Test cache initialization with different parameters."""
        test_cases = [
            (1, False, False),
            (4, False, True),
            (1, True, False),
        ]

        for page_size, disable, enable_events in test_cases:
            with self.subTest(
                page_size=page_size, disable=disable, enable_events=enable_events
            ):
                cache = RadixCache.create_simulated(
                    disable=disable,
                    page_size=page_size,
                    enable_kv_cache_events=enable_events,
                )

                self.assertEqual(cache.page_size, page_size)
                self.assertEqual(cache.disable, disable)
                self.assertEqual(cache.enable_kv_cache_events, enable_events)
                self.assertEqual(cache.device, torch.device("cpu"))
                self.assertIsNotNone(cache.root_node)
                self.assertEqual(len(cache.root_node.key), 0)
```
**EN:** Test cache initialization with different parameters. This test exercises `test_init_variations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test cache initialization with different parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_variations`。

### Lines 270-287: test case reset / 测试用例 reset
```python
    def test_reset(self):
        """Test reset method."""
        cache = RadixCache.create_simulated()

        # Insert some data
        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3]),
                value=torch.tensor([10, 20, 30], dtype=torch.int64),
            )
        )
        self.assertGreater(cache.total_size(), 0)

        # Reset
        cache.reset()
        self.assertEqual(cache.total_size(), 0)
        self.assertEqual(cache.evictable_size(), 0)
        self.assertEqual(cache.protected_size(), 0)
```
**EN:** Test reset method. This test exercises `test_reset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test reset method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_reset`。

### Lines 289-319: test case insert and match basic / 测试用例 insert and match basic
```python
    def test_insert_and_match_basic(self):
        """Test basic insert and match operations."""
        for disable_cache in [False, True]:
            with self.subTest(disable_cache=disable_cache):
                cache = RadixCache.create_simulated(disable=disable_cache)

                key = RadixKey([1, 2, 3])
                value = torch.tensor([10, 20, 30], dtype=torch.int64)
                result = cache.insert(InsertParams(key=key, value=value))
                prefix_len = result.prefix_len

                if disable_cache:
                    self.assertEqual(prefix_len, 0)
                    self.assertEqual(cache.total_size(), 0)
                    continue

                self.assertEqual(prefix_len, 0)  # No existing prefix
                self.assertEqual(cache.total_size(), 3)
                self.assertEqual(cache.evictable_size(), 3)

                # Test match_prefix
                result = cache.match_prefix(MatchPrefixParams(key=RadixKey([1, 2, 3])))
                self.assertEqual(len(result.device_indices), 3)
                torch.testing.assert_close(result.device_indices, value)

                # Test partial match
                result = cache.match_prefix(MatchPrefixParams(key=RadixKey([1, 2])))
                self.assertEqual(len(result.device_indices), 2)
                torch.testing.assert_close(
                    result.device_indices, torch.tensor([10, 20], dtype=torch.int64)
                )
```
**EN:** Test basic insert and match operations. This test exercises `test_insert_and_match_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic insert and match operations. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_insert_and_match_basic`。

### Lines 321-331: test case insert with none value / 测试用例 insert with none value
```python
    def test_insert_with_none_value(self):
        """Test insert with None value (should use token_ids as list)."""
        cache = RadixCache.create_simulated()

        key = RadixKey([1, 2, 3])
        result = cache.insert(InsertParams(key=key, value=None))
        prefix_len = result.prefix_len

        # When None is passed, it should create value from token_ids
        self.assertEqual(prefix_len, 0)
        self.assertEqual(cache.total_size(), 3)
```
**EN:** Test insert with None value (should use token_ids as list). This test exercises `test_insert_with_none_value` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test insert with None value (should use token_ids as list). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_insert_with_none_value`。

### Lines 333-352: test case total size / 测试用例 total size
```python
    def test_total_size(self):
        """Test total_size calculation."""
        cache = RadixCache.create_simulated()

        self.assertEqual(cache.total_size(), 0)

        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3]),
                value=torch.tensor([10, 20, 30], dtype=torch.int64),
            )
        )
        self.assertEqual(cache.total_size(), 3)

        cache.insert(
            InsertParams(
                key=RadixKey([4, 5]), value=torch.tensor([40, 50], dtype=torch.int64)
            )
        )
        self.assertEqual(cache.total_size(), 5)
```
**EN:** Test total_size calculation. This test exercises `test_total_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test total_size calculation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_total_size`。

### Lines 354-384: test case kv cache events / 测试用例 kv cache events
```python
    def test_kv_cache_events(self):
        """Test KV cache events functionality."""
        test_cases = [
            (1, True),
            (2, True),
            (1, False),
        ]

        for page_size, enable_events in test_cases:
            with self.subTest(page_size=page_size, enable_events=enable_events):
                cache = RadixCache.create_simulated(
                    page_size=page_size, enable_kv_cache_events=enable_events
                )

                # Insert data
                cache.insert(InsertParams(key=RadixKey([1, 2, 3, 4, 5]), value=None))

                # Take events
                events = cache.take_events()

                if enable_events:
                    self.assertGreater(len(events), 0)
                    # Verify events include BlockStored events (there might be other event types)
                    block_stored_events = [
                        e for e in events if isinstance(e, BlockStored)
                    ]
                    self.assertGreater(len(block_stored_events), 0)
                    for event in block_stored_events:
                        self.assertLessEqual(len(event.token_ids), page_size)
                else:
                    self.assertEqual(len(events), 0)
```
**EN:** Test KV cache events functionality. This test exercises `test_kv_cache_events` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test KV cache events functionality. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kv_cache_events`。

### Lines 386-421: test case kv cache events with eviction / 测试用例 kv cache events with eviction
```python
    def test_kv_cache_events_with_eviction(self):
        """Test KV cache events include removal events."""
        mock_allocator = unittest.mock.Mock()
        mock_allocator.device = torch.device("cpu")

        cache = RadixCache.create_simulated(
            mock_allocator=mock_allocator, enable_kv_cache_events=True
        )

        # Insert and then evict data
        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3]),
                value=torch.tensor([10, 20, 30], dtype=torch.int64),
            )
        )
        result = cache.evict(EvictParams(num_tokens=3))
        self.assertIsInstance(result, EvictResult)
        self.assertGreaterEqual(
            result.num_tokens_evicted,
            3,
            f"evicted {result.num_tokens_evicted} tokens, expected at least 3",
        )

        # Take events - should include both store and remove events
        events = cache.take_events()
        self.assertGreater(len(events), 0)

        # Check event types
        event_types = [type(event).__name__ for event in events]
        self.assertIn("BlockStored", event_types)

        # Verify BlockRemoved event content
        remove_events = [e for e in events if isinstance(e, BlockRemoved)]
        for event in remove_events:
            self.assertGreater(len(event.block_hashes), 0)
```
**EN:** Test KV cache events include removal events. This test exercises `test_kv_cache_events_with_eviction` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test KV cache events include removal events. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kv_cache_events_with_eviction`。

### Lines 423-472: test case extra key isolation / 测试用例 extra key isolation
```python
    def test_extra_key_isolation(self):
        """Test that keys with different extra_key values are isolated."""
        cache = RadixCache.create_simulated()

        # Insert same token sequence with different extra keys
        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3], "key1"),
                value=torch.tensor([10, 20, 30], dtype=torch.int64),
            )
        )
        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3], "key2"),
                value=torch.tensor([40, 50, 60], dtype=torch.int64),
            )
        )
        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3], None),
                value=torch.tensor([70, 80, 90], dtype=torch.int64),
            )
        )

        # Keys with different extra_key should not match each other
        result1 = cache.match_prefix(MatchPrefixParams(key=RadixKey([1, 2, 3], "key1")))
        result2 = cache.match_prefix(MatchPrefixParams(key=RadixKey([1, 2, 3], "key2")))
        result3 = cache.match_prefix(MatchPrefixParams(key=RadixKey([1, 2, 3], None)))
        result4 = cache.match_prefix(
            MatchPrefixParams(key=RadixKey([1, 2, 3], "nonexistent"))
        )

        # Each should match only its own data
        self.assertEqual(len(result1.device_indices), 3)
        torch.testing.assert_close(
            result1.device_indices, torch.tensor([10, 20, 30], dtype=torch.int64)
        )

        self.assertEqual(len(result2.device_indices), 3)
        torch.testing.assert_close(
            result2.device_indices, torch.tensor([40, 50, 60], dtype=torch.int64)
        )

        self.assertEqual(len(result3.device_indices), 3)
        torch.testing.assert_close(
            result3.device_indices, torch.tensor([70, 80, 90], dtype=torch.int64)
        )

        # Non-existent extra_key should not match
        self.assertEqual(len(result4.device_indices), 0)
```
**EN:** Test that keys with different extra_key values are isolated. This test exercises `test_extra_key_isolation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that keys with different extra_key values are isolated. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extra_key_isolation`。

### Lines 474-501: test case lock ref operations / 测试用例 lock ref operations
```python
    def test_lock_ref_operations(self):
        """Test lock reference counting operations."""
        cache = RadixCache.create_simulated()

        # Insert sequence
        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3]),
                value=torch.tensor([10, 20, 30], dtype=torch.int64),
            )
        )

        # Get node
        result = cache.match_prefix(MatchPrefixParams(key=RadixKey([1, 2, 3])))
        node = result.last_device_node

        initial_evictable = cache.evictable_size()
        initial_protected = cache.protected_size()

        # Lock the node
        cache.inc_lock_ref(node)
        self.assertEqual(cache.protected_size(), initial_protected + 3)
        self.assertEqual(cache.evictable_size(), initial_evictable - 3)

        # Unlock the node
        cache.dec_lock_ref(node)
        self.assertEqual(cache.protected_size(), initial_protected)
        self.assertEqual(cache.evictable_size(), initial_evictable)
```
**EN:** Test lock reference counting operations. This test exercises `test_lock_ref_operations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test lock reference counting operations. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lock_ref_operations`。

### Lines 503-535: test case evict functionality / 测试用例 evict functionality
```python
    def test_evict_functionality(self):
        """Test eviction functionality."""
        mock_allocator = unittest.mock.Mock()
        mock_allocator.device = torch.device("cpu")

        cache = RadixCache.create_simulated(mock_allocator=mock_allocator)

        # Insert sequences
        cache.insert(
            InsertParams(
                key=RadixKey([1, 2]), value=torch.tensor([10, 20], dtype=torch.int64)
            )
        )
        cache.insert(
            InsertParams(
                key=RadixKey([3, 4]), value=torch.tensor([30, 40], dtype=torch.int64)
            )
        )

        initial_size = cache.total_size()

        # Evict some tokens
        result = cache.evict(EvictParams(num_tokens=2))
        self.assertIsInstance(result, EvictResult)
        self.assertGreaterEqual(
            result.num_tokens_evicted,
            2,
            f"evicted {result.num_tokens_evicted} tokens, expected at least 2",
        )

        # Should have called free and reduced size
        mock_allocator.free.assert_called()
        self.assertLess(cache.total_size(), initial_size)
```
**EN:** Test eviction functionality. This test exercises `test_evict_functionality` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test eviction functionality. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_evict_functionality`。

### Lines 537-563: test case page alignment boundary / 测试用例 page alignment boundary
```python
    def test_page_alignment_boundary(self):
        """Test page alignment with different sizes."""
        test_cases = [
            (1, 5),
            (2, 5),
            (4, 6),
        ]

        for page_size, sequence_length in test_cases:
            with self.subTest(page_size=page_size, sequence_length=sequence_length):
                cache = RadixCache.create_simulated(page_size=page_size)

                tokens = list(range(sequence_length))
                key = RadixKey(tokens)
                cache.insert(
                    InsertParams(
                        key=key,
                        value=torch.tensor(tokens, dtype=torch.int64)[: len(key)],
                    )
                )

                result = cache.match_prefix(MatchPrefixParams(key=RadixKey(tokens)))
                self.assertGreater(len(result.device_indices), 0)

                # Match length should be page-aligned
                match_len = len(result.device_indices)
                self.assertEqual(match_len % page_size, 0)
```
**EN:** Test page alignment with different sizes. This test exercises `test_page_alignment_boundary` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test page alignment with different sizes. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_page_alignment_boundary`。

### Lines 565-580: test case pretty print basic / 测试用例 pretty print basic
```python
    def test_pretty_print_basic(self):
        """Test pretty_print produces output."""
        cache = RadixCache.create_simulated()

        cache.insert(
            InsertParams(
                key=RadixKey([1, 2, 3]),
                value=torch.tensor([10, 20, 30], dtype=torch.int64),
            )
        )

        # Just test that it doesn't crash
        try:
            cache.pretty_print()
        except Exception as e:
            self.fail(f"pretty_print raised an exception: {e}")
```
**EN:** Test pretty_print produces output. This test exercises `test_pretty_print_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test pretty_print produces output. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pretty_print_basic`。

### Lines 582-601: test case all values flatten / 测试用例 all values flatten
```python
    def test_all_values_flatten(self):
        """Test all_values_flatten method."""
        cache = RadixCache.create_simulated()

        cache.insert(
            InsertParams(
                key=RadixKey([1, 2]), value=torch.tensor([10, 20], dtype=torch.int64)
            )
        )
        cache.insert(
            InsertParams(
                key=RadixKey([3, 4]), value=torch.tensor([30, 40], dtype=torch.int64)
            )
        )

        all_values = cache.all_values_flatten()
        self.assertEqual(len(all_values), 4)
        # Values should contain all inserted values (order may vary)
        values_set = set(all_values.tolist())
        self.assertEqual(values_set, {10, 20, 30, 40})
```
**EN:** Test all_values_flatten method. This test exercises `test_all_values_flatten` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test all_values_flatten method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_values_flatten`。

### Lines 603-646: test case advanced prefix match with node splits / 测试用例 advanced prefix match with node splits
```python
    def test_advanced_prefix_match_with_node_splits(self):
        """Advanced prefix matching: splits inside nodes and across pages."""
        for page_size in [1, 2]:
            with self.subTest(page_size=page_size):
                cache = RadixCache.create_simulated(page_size=page_size)

                # Insert a long sequence that will be split later.
                seq1 = [1, 2, 3, 4, 5, 6, 7, 8]
                val1 = torch.tensor([x * 10 for x in seq1], dtype=torch.int64)
                cache.insert(InsertParams(key=RadixKey(seq1), value=val1))

                # Insert a diverging branch to create an internal node on the path.
                seq2 = [1, 2, 9, 10]
                val2 = torch.tensor([x * 10 for x in seq2], dtype=torch.int64)
                cache.insert(InsertParams(key=RadixKey(seq2), value=val2))
                print(cache.pretty_print())

                baseline_total = cache.total_size()
                expected_total = 10  # 8 + 2
                self.assertEqual(baseline_total, expected_total)

                # Match that causes a split inside an existing node:
                # take first 4 tokens of seq1, then diverge.
                query1 = [1, 2, 3, 4, 999, 1000]
                result1 = cache.match_prefix(MatchPrefixParams(key=RadixKey(query1)))
                torch.testing.assert_close(result1.device_indices, val1[:4])
                # No data change after structural split during matching.
                self.assertEqual(cache.total_size(), baseline_total)

                # Full match of the long sequence still returns the full indices.
                result_full = cache.match_prefix(MatchPrefixParams(key=RadixKey(seq1)))
                torch.testing.assert_close(result_full.device_indices, val1)

                # Another split deeper on the path (after matching 6 tokens, then diverge).
                query2 = [1, 2, 3, 4, 5, 6, 777, 888]
                result2 = cache.match_prefix(MatchPrefixParams(key=RadixKey(query2)))
                torch.testing.assert_close(result2.device_indices, val1[:6])
                self.assertEqual(cache.total_size(), baseline_total)

                # Matching the short diverging branch should return exactly its indices.
                result_branch = cache.match_prefix(
                    MatchPrefixParams(key=RadixKey(seq2))
                )
                torch.testing.assert_close(result_branch.device_indices, val2)
```
**EN:** Advanced prefix matching: splits inside nodes and across pages. This test exercises `test_advanced_prefix_match_with_node_splits` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Advanced prefix matching: splits inside nodes and across pages. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_advanced_prefix_match_with_node_splits`。

### Lines 648-672: test case hash value storage / 测试用例 hash value storage
```python
    def test_hash_value_storage(self):
        """Test that hash_value is stored correctly after insert operations."""
        cache = RadixCache.create_simulated(
            page_size=4,
            enable_kv_cache_events=True,
        )

        # Insert a sequence
        cache.insert(InsertParams(key=RadixKey([1, 2, 3, 4, 5, 6, 7, 8]), value=None))

        # Trigger event emission to compute hash_value lazily
        cache.take_events()

        # Find the inserted node (traverse from root)
        node = cache.root_node
        for i in range(0, 8, 4):  # page_size=4, so 2 pages
            child_key = tuple([1, 2, 3, 4][:4]) if i == 0 else tuple([5, 6, 7, 8][:4])
            if child_key in node.children:
                node = node.children[child_key]
                break

        # Verify hash_value is set (computed lazily during event emission)
        self.assertIsNotNone(node.hash_value)
        # Should have 2 pages (8 tokens / 4 page_size)
        self.assertEqual(len(node.hash_value), 2)
```
**EN:** Test that hash_value is stored correctly after insert operations. This test exercises `test_hash_value_storage` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that hash_value is stored correctly after insert operations. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hash_value_storage`。

### Lines 674-706: test case hash value repeating tokens / 测试用例 hash value repeating tokens
```python
    def test_hash_value_repeating_tokens(self):
        """Test that repeating token patterns get different hash values."""
        cache = RadixCache.create_simulated(
            page_size=4,
            enable_kv_cache_events=True,
        )

        # Insert a sequence with repeating token pattern: [1,2,3,4, 1,2,3,4]
        cache.insert(InsertParams(key=RadixKey([1, 2, 3, 4, 1, 2, 3, 4]), value=None))

        events = cache.take_events()
        block_stored_events = [e for e in events if isinstance(e, BlockStored)]

        # Should have 2 blocks (2 pages of size 4)
        self.assertEqual(len(block_stored_events), 2)

        # Extract block hashes
        block_hash_1 = block_stored_events[0].block_hashes[0]
        block_hash_2 = block_stored_events[1].block_hashes[0]

        # The two blocks should have DIFFERENT hashes despite same content
        # because they are at different positions (sequence-aware hashing)
        self.assertNotEqual(
            block_hash_1,
            block_hash_2,
            "Repeating token patterns should get different sequence-aware hashes",
        )

        # First block should have no parent
        self.assertIsNone(block_stored_events[0].parent_block_hash)

        # Second block's parent should be the first block's hash
        self.assertEqual(block_stored_events[1].parent_block_hash, block_hash_1)
```
**EN:** Test that repeating token patterns get different hash values. This test exercises `test_hash_value_repeating_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that repeating token patterns get different hash values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hash_value_repeating_tokens`。

### Lines 708-734: test case hash value split / 测试用例 hash value split
```python
    def test_hash_value_split(self):
        """Test that hash_value is split correctly when nodes are split."""
        cache = RadixCache.create_simulated(
            page_size=2,
            enable_kv_cache_events=True,
        )

        # Insert a sequence that will cause a split
        cache.insert(InsertParams(key=RadixKey([1, 2, 3, 4]), value=None))
        cache.take_events()  # Clear events and compute hash_value for first node

        # Insert a diverging sequence that will cause a split at page boundary
        cache.insert(InsertParams(key=RadixKey([1, 2, 5, 6]), value=None))
        cache.take_events()  # Trigger event emission to compute hash_value

        # Find the split node
        node = cache.root_node
        child_key = tuple([1, 2])
        if child_key in node.children:
            node = node.children[child_key]
            # After split and event emission, hash_value should be computed
            # Note: If hash_value wasn't set before split, it will be computed lazily
            # during event emission. If it was set, it will be split.
            # Either way, after events are emitted, it should be set.
            self.assertIsNotNone(node.hash_value)
            # Should have 1 page (split at page_size=2)
            self.assertEqual(len(node.hash_value), 1)
```
**EN:** Test that hash_value is split correctly when nodes are split. This test exercises `test_hash_value_split` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that hash_value is split correctly when nodes are split. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hash_value_split`。

### Lines 736-767: test case memory allocated / 测试用例 memory allocated
```python
    def test_memory_allocated(self):
        keys, values = [], []

        num_seqs = 10000
        vocab_size = 1000
        base_prefix_len = 10000
        suffix_len = 100

        torch_allocated_before = torch.cuda.memory_allocated()

        # build dataset with common prefix
        common_prefix = [random.randint(1, vocab_size) for _ in range(base_prefix_len)]
        for _ in range(num_seqs):
            suffix = [random.randint(1, vocab_size) for _ in range(suffix_len)]
            seq = common_prefix + suffix
            keys.append(seq)
            values.append(torch.zeros(len(seq), device="cuda", dtype=torch.int32))

        cache: RadixCache = RadixCache.create_simulated()

        for key, value in zip(keys, values):
            cache.insert(InsertParams(key=RadixKey(key), value=value))

        del values

        torch_allocated = torch.cuda.memory_allocated() - torch_allocated_before
        cache_size_bytes = cache.total_size() * 4
        print(f"\nCache size (MB): {cache_size_bytes / (1024 * 1024)}")
        print(f"Torch allocated (MB): {torch_allocated / (1024 * 1024)}")

        # The cache size should be within reasonable bounds of the actual allocated memory.
        self.assertLess(torch_allocated, cache_size_bytes * 2)
```
**EN:** This test exercises `test_memory_allocated` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_allocated`。

### Lines 769-775: test case available and evictable str / 测试用例 available and evictable str
```python
    def test_available_and_evictable_str(self):
        mock_allocator = unittest.mock.Mock()
        mock_allocator.available_size.return_value = 10
        cache: RadixCache = RadixCache.create_simulated(mock_allocator=mock_allocator)

        print(cache.available_and_evictable_str())
        print(available_and_evictable_str(cache))
```
**EN:** This test exercises `test_available_and_evictable_str` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_available_and_evictable_str`。

### Lines 778-779: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestRadixKey`: Test cases for RadixKey class. / 用于组织相关测试、夹具或辅助方法。
- `TestTreeNode`: Test cases for TreeNode class. / 用于组织相关测试、夹具或辅助方法。
- `TestRadixCache`: Test cases for RadixCache class. / 用于组织相关测试、夹具或辅助方法。
- `TestRadixKey.test_init_basic`: Test basic initialization of RadixKey. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_basic`。
- `TestRadixKey.test_init_with_extra_key`: Test initialization with extra_key. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_with_extra_key`。
- `TestRadixKey.test_len`: Test __len__ method. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_len`。
- `TestRadixKey.test_iter`: Test __iter__ method. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_iter`。
- `TestRadixKey.test_len_and_iter`: Test __len__ and __iter__ methods. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_len_and_iter`。
- `TestRadixKey.test_getitem_int`: Test __getitem__ with int index. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getitem_int`。
- `TestRadixKey.test_getitem_slice`: Test __getitem__ with slice and edge cases. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getitem_slice`。
- `TestRadixKey.test_getitem_invalid_index`: Test __getitem__ with invalid indices. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getitem_invalid_index`。
- `TestRadixKey.test_repr`: Test __repr__ method. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repr`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `time`, `unittest`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.mem_cache.common`, `sglang.test.ci.ci_register`, `sglang.srt.disaggregation.kv_events`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.radix_cache`

- **Total lines / 总行数**: 779
