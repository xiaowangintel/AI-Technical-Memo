# test_radix_cache_slru_accuracy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_radix_cache_slru_accuracy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates radix cache slru accuracy behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 radix cache slru accuracy 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.mem_cache.allocator import TokenToKVPoolAllocator
from sglang.srt.mem_cache.base_prefix_cache import (
    EvictParams,
    InsertParams,
    MatchPrefixParams,
)
from sglang.srt.mem_cache.cache_init_params import CacheInitParams
from sglang.srt.mem_cache.memory_pool import MHATokenToKVPool, ReqToTokenPool
from sglang.srt.mem_cache.radix_cache import RadixCache, RadixKey
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_prefix_cache`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_prefix_cache`。

### Lines 16-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=8, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=8, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-21: class TestSLRUAccuracy declaration / 类 TestSLRUAccuracy 声明
```python
class TestSLRUAccuracy(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 22-60: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Setup minimal memory pools for testing"""
        torch.set_default_device(None)
        device = "cpu"
        dtype = torch.float16

        # Create smaller KV cache to ensure evictions occur
        self.kv_cache = MHATokenToKVPool(
            size=8,  # Very small size to trigger evictions quickly
            page_size=1,
            dtype=dtype,
            head_num=8,
            head_dim=64,
            layer_num=1,
            device=device,
            enable_memory_saver=False,
        )

        # Create token-to-KV pool allocator
        self.token_to_kv_pool = TokenToKVPoolAllocator(
            size=8, dtype=dtype, device=device, kvcache=self.kv_cache, need_sort=False
        )

        # Create req-to-token pool
        self.req_to_token_pool = ReqToTokenPool(
            size=8, max_context_len=1024, device=device, enable_memory_saver=False
        )

        # Create a cache with the memory pools
        params = CacheInitParams(
            disable=False,
            req_to_token_pool=self.req_to_token_pool,
            token_to_kv_pool_allocator=self.token_to_kv_pool,
            page_size=1,
            eviction_policy="slru",
            enable_kv_cache_events=False,
        )

        self.cache = RadixCache(params)
```
**EN:** Setup minimal memory pools for testing This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Setup minimal memory pools for testing 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 62-133: test case eviction mechanism / 测试用例 eviction mechanism
```python
    def test_eviction_mechanism(self):
        """Test that SLRU eviction mechanism works correctly"""

        # Insert one key-value three times (high frequency access)
        frequent_key = RadixKey([1, 2])  # High hit rate, should be retained
        frequent_val = torch.tensor([10, 20], dtype=torch.int64)

        # Insert the frequent key multiple times to increase its hit count
        for _ in range(3):
            self.cache.insert(InsertParams(key=frequent_key, value=frequent_val))

        # Insert first low-frequency key-value pair that should be evicted
        first_low_freq_key = RadixKey([5, 6])  # Low hit rate, should be evicted
        first_low_freq_val = torch.tensor([50, 60], dtype=torch.int64)

        self.cache.insert(
            InsertParams(key=first_low_freq_key, value=first_low_freq_val)
        )

        # Insert other key-values once each (low frequency access) - fill up the cache
        other_keys = []
        for i in range(4):  # Reduce the number to fit in our smaller cache
            key = RadixKey([i + 10])  # Unique keys for low-frequency items
            val = torch.tensor([i + 100], dtype=torch.int64)
            self.cache.insert(InsertParams(key=key, value=val))
            other_keys.append(key)

        # Now insert more items to trigger evictions
        for i in range(6, 10):  # Add more items to definitely exceed capacity
            key = RadixKey([i * 2])  # Different pattern to avoid conflicts
            val = torch.tensor([i * 200], dtype=torch.int64)
            self.cache.insert(InsertParams(key=key, value=val))

        # Now trigger eviction explicitly to make space
        evict_result = self.cache.evict(
            EvictParams(num_tokens=4)
        )  # Try to evict 4 tokens worth of space

        # Check if the frequently accessed key-value is still present
        # The frequent key should have higher hit count and remain in cache due to SLRU policy
        frequent_match_result = self.cache.match_prefix(
            MatchPrefixParams(key=frequent_key)
        )

        # Check if the first low-frequency key-value has been evicted
        # The first low-freq key should have lower hit count and be evicted due to SLRU policy
        first_low_freq_match_result = self.cache.match_prefix(
            MatchPrefixParams(key=first_low_freq_key)
        )

        # Verify the frequent key is still present in cache after evictions
        self.assertIsNotNone(
            frequent_match_result,
            "Frequently accessed key should still be in cache after evictions",
        )

        # Check if the tensor is empty, which indicates the key was not found (evicted)
        is_frequent_key_present = frequent_match_result.device_indices.numel() > 0
        self.assertTrue(
            is_frequent_key_present,
            "Frequently accessed key should still be in cache after evictions",
        )

        # Verify the first low-frequency key has been evicted
        # The device_indices tensor should be empty when the key is not found
        is_first_low_freq_key_present = (
            first_low_freq_match_result.device_indices.numel() > 0
        )
        self.assertFalse(
            is_first_low_freq_key_present,
            "First inserted low-frequency key should be evicted after evictions",
        )
```
**EN:** Test that SLRU eviction mechanism works correctly This test exercises `test_eviction_mechanism` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that SLRU eviction mechanism works correctly 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eviction_mechanism`。

### Lines 136-137: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestSLRUAccuracy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSLRUAccuracy.setUp`: Setup minimal memory pools for testing / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSLRUAccuracy.test_eviction_mechanism`: Test that SLRU eviction mechanism works correctly / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eviction_mechanism`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.radix_cache`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 137
