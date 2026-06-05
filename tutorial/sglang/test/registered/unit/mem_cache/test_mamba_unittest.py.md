# test_mamba_unittest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_mamba_unittest.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mamba unittest behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 mamba unittest 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.configs.mamba_utils import Mamba2CacheParams, Mamba2StateShape
from sglang.srt.disaggregation.kv_events import BlockRemoved, BlockStored
from sglang.srt.environ import envs
from sglang.srt.managers.schedule_batch import Req
from sglang.srt.mem_cache.allocator import TokenToKVPoolAllocator
from sglang.srt.mem_cache.base_prefix_cache import (
    EvictParams,
    InsertParams,
    MatchPrefixParams,
)
from sglang.srt.mem_cache.cache_init_params import CacheInitParams
from sglang.srt.mem_cache.common import available_and_evictable_str
from sglang.srt.mem_cache.hi_mamba_radix_cache import HiMambaRadixCache
from sglang.srt.mem_cache.mamba_radix_cache import LRUList, MambaRadixCache, TreeNode
from sglang.srt.mem_cache.memory_pool import HybridLinearKVPool, HybridReqToTokenPool
from sglang.srt.mem_cache.radix_cache import RadixKey
from sglang.srt.sampling.sampling_params import SamplingParams
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.configs.mamba_utils`, `sglang.srt.disaggregation.kv_events`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.configs.mamba_utils`, `sglang.srt.disaggregation.kv_events`。

### Lines 26-27: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=10, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=9, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 30-30: class TestMamba declaration / 类 TestMamba 声明
```python
class TestMamba(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 31-33: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        pass
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 35-37: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        pass
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 39-68: test case hybrid linear kv pool / 测试用例 hybrid linear kv pool
```python
    def test_hybrid_linear_kv_pool(self):
        size = 16
        head_num = 2
        head_dim = 256
        num_layers = 48
        global_interval = 4
        dtype = torch.bfloat16
        device = get_device()
        full_attention_layer_ids = [
            i for i in range(global_interval - 1, num_layers, global_interval)
        ]
        pool = HybridLinearKVPool(
            size=size,
            dtype=dtype,
            page_size=1,
            head_num=head_num,
            head_dim=head_dim,
            full_attention_layer_ids=full_attention_layer_ids,
            enable_kvcache_transpose=False,
            device=device,
            enable_memory_saver=False,
            mamba_pool=None,
        )
        assert pool._transfer_full_attention_id(global_interval - 1) == 0
        assert pool._transfer_full_attention_id(2 * global_interval - 1) == 1
        with self.assertRaises(ValueError) as context:
            pool._transfer_full_attention_id(1)
        self.assertIn(
            "layer_id=1 not in full attention layers:", str(context.exception)
        )
```
**EN:** This test exercises `test_hybrid_linear_kv_pool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hybrid_linear_kv_pool`。

### Lines 70-144: test case mamba pool / 测试用例 mamba pool
```python
    def test_mamba_pool(self):
        max_num_reqs = 10
        mamba_cache_size = 20
        max_context_len = 128
        device = get_device()
        global_interval = 4
        num_layers = 48
        full_attention_layer_ids = [
            i for i in range(global_interval - 1, num_layers, global_interval)
        ]
        mamba_layers = [
            i for i in range(num_layers) if i not in full_attention_layer_ids
        ]
        shape = Mamba2StateShape.create(
            tp_world_size=1,
            intermediate_size=4096,
            n_groups=16,
            num_heads=32,
            head_dim=128,
            state_size=128,
            conv_kernel=4,
        )

        with envs.SGLANG_MAMBA_SSM_DTYPE.override("bfloat16"):
            mamba2_cache_params = Mamba2CacheParams(shape=shape, layers=mamba_layers)

        req_to_token_pool = HybridReqToTokenPool(
            size=max_num_reqs,
            mamba_size=mamba_cache_size,
            mamba_spec_state_size=max_num_reqs,
            max_context_len=max_context_len,
            device=device,
            enable_memory_saver=False,
            cache_params=mamba2_cache_params,
            mamba_layer_ids=mamba_layers,
            enable_mamba_extra_buffer=False,
            speculative_num_draft_tokens=3,
        )

        assert req_to_token_pool.available_size() == max_num_reqs
        assert req_to_token_pool.mamba_pool.available_size() == mamba_cache_size

        sampling_params = SamplingParams(
            temperature=0,
            max_new_tokens=1,
        )
        req = Req(
            rid=0,
            origin_input_text="",
            origin_input_ids=[],
            sampling_params=sampling_params,
        )

        # alloc req
        req_to_token_pool.alloc([req])
        assert req_to_token_pool.available_size() == max_num_reqs - 1
        assert req_to_token_pool.mamba_pool.available_size() == mamba_cache_size - 1

        # free req
        req_to_token_pool.free_mamba_cache(req)
        req_to_token_pool.free(req)
        assert req_to_token_pool.available_size() == max_num_reqs
        assert req_to_token_pool.mamba_pool.available_size() == mamba_cache_size

        # alloc req without free mamba cache
        req.mamba_pool_idx = None
        req_to_token_pool.alloc([req])
        req_to_token_pool.free(req)
        assert req_to_token_pool.available_size() == max_num_reqs
        assert req_to_token_pool.mamba_pool.available_size() == mamba_cache_size - 1

        # alloc again
        req_to_token_pool.alloc([req])
        assert req_to_token_pool.available_size() == max_num_reqs - 1
        assert req_to_token_pool.mamba_pool.available_size() == mamba_cache_size - 1
```
**EN:** This test exercises `test_mamba_pool` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_pool`。

### Lines 146-225: test case mamba radix cache 1 (part 1/3) / 测试用例 mamba radix cache 1（第 1/3 部分）
```python
    def test_mamba_radix_cache_1(self):
        tree, allocator, req_to_token_pool, make_dummy_req = (
            self._setup_tree_and_allocator()
        )
        mamba_pool = req_to_token_pool.mamba_pool
        # test
        print(
            f"[Start] allocator mamba available size: {mamba_pool.available_size()}, full available size: {allocator.available_size()}"
        )
        req1 = make_dummy_req()
        req1_token_ids, req1_kv_indices = [1, 2, 3], allocator.alloc(3)
        assert len(req1_token_ids) == len(req1_kv_indices)
        print(
            f"req1: inserting, req1_token_ids: {req1_token_ids}, req1_kv_indices: {req1_kv_indices}"
        )
        key = RadixKey(req1_token_ids)
        result = tree.insert(
            InsertParams(
                key=key,
                value=req1_kv_indices[: len(key)],
                mamba_value=req1.mamba_pool_idx.unsqueeze(0),
            )
        )
        prefix_len = result.prefix_len
        print(
            f"req1: prefix_len: {prefix_len}, allocator mamba available size: {mamba_pool.available_size()}, full available size: {allocator.available_size()}"
        )
        req2 = make_dummy_req()
        req2_token_ids, req2_kv_indices = [1, 2, 3, 4, 5, 6, 7], allocator.alloc(7)
        assert len(req2_token_ids) == len(req2_kv_indices)
        print(
            f"req2: inserting, req2_token_ids: {req2_token_ids}, req2_kv_indices: {req2_kv_indices}"
        )
        key = RadixKey(req2_token_ids)
        result = tree.insert(
            InsertParams(
                key=key,
                value=req2_kv_indices[: len(key)],
                mamba_value=req2.mamba_pool_idx.unsqueeze(0),
            )
        )
        prefix_len = result.prefix_len
        print(
            f"req2: prefix_len: {prefix_len}, allocator mamba available size: {mamba_pool.available_size()}, full available size: {allocator.available_size()}"
        )

        req3 = make_dummy_req()
        req3_token_ids, req3_kv_indices = [10, 11, 12], allocator.alloc(3)
        assert len(req3_token_ids) == len(req3_kv_indices)
        print(
            f"req3: inserting, req3_token_ids: {req3_token_ids}, req3_kv_indices: {req3_kv_indices}"
        )
        key = RadixKey(req3_token_ids)
        result = tree.insert(
            InsertParams(
                key=key,
                value=req3_kv_indices[: len(key)],
                mamba_value=req3.mamba_pool_idx.unsqueeze(0),
            )
        )
        prefix_len = result.prefix_len
        print(
            f"req3: prefix_len: {prefix_len}, allocator mamba available size: {mamba_pool.available_size()}, full available size: {allocator.available_size()}"
        )
        req4 = make_dummy_req()
        req4_token_ids, req4_kv_indices = [1, 2, 3, 4, 5, 60, 70], allocator.alloc(7)
        assert len(req4_token_ids) == len(req4_kv_indices)
        print(
            f"req4: inserting, req4_token_ids: {req4_token_ids}, req4_kv_indices: {req4_kv_indices}"
        )
        key = RadixKey(req4_token_ids)
        result = tree.insert(
            InsertParams(
                key=key,
                value=req4_kv_indices[: len(key)],
                mamba_value=req4.mamba_pool_idx.unsqueeze(0),
            )
        )
        prefix_len = result.prefix_len
        print(
```
**EN:** This test exercises `test_mamba_radix_cache_1` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_1`。 这一段对应同一逻辑块的第 1 部分。

### Lines 226-305: test case mamba radix cache 1 (part 2/3) / 测试用例 mamba radix cache 1（第 2/3 部分）
```python
            f"req4: prefix_len: {prefix_len}, allocator mamba available size: {mamba_pool.available_size()}, full available size: {allocator.available_size()}"
        )

        tree.pretty_print()
        full_num_tokens = 1
        print(f"evicting {full_num_tokens} full token")
        result = tree.evict(EvictParams(num_tokens=full_num_tokens))
        assert (
            result.num_tokens_evicted >= full_num_tokens
        ), f"evicted {result.num_tokens_evicted} full tokens, expected {full_num_tokens}"
        tree.pretty_print()

        mamba_num = 1
        print(f"evicting {mamba_num} mamba")
        result = tree.evict(EvictParams(num_tokens=0, mamba_num=mamba_num))
        assert (
            result.mamba_num_evicted >= mamba_num
        ), f"evicted {result.mamba_num_evicted} mamba states, expected {mamba_num}"
        tree.pretty_print()

        req5_token_ids = [1, 2, 3, 4, 5]
        result = tree.match_prefix(MatchPrefixParams(key=RadixKey(req5_token_ids)))
        kv_indices, last_node = result.device_indices, result.last_device_node
        print(
            f"req5: token_ids: {req5_token_ids}, matched kv_indices: {kv_indices}, last_node.key: {last_node.key}"
        )
        assert len(kv_indices) == 0

        req6_token_ids = [1, 2, 3, 4, 5, 60, 70]
        result = tree.match_prefix(MatchPrefixParams(key=RadixKey(req6_token_ids)))
        kv_indices, last_node = result.device_indices, result.last_device_node
        print(
            f"req6: token_ids: {req6_token_ids}, matched kv_indices: {kv_indices}, last_node.key: {last_node.key}"
        )
        assert len(kv_indices) == 7
        assert len(last_node.key) == 2

        req7_token_ids = [1, 2, 3, 4, 5, 6, 7]
        result = tree.match_prefix(MatchPrefixParams(key=RadixKey(req7_token_ids)))
        kv_indices, last_node = result.device_indices, result.last_device_node
        print(
            f"req7: token_ids: {req7_token_ids}, matched kv_indices: {kv_indices}, last_node.key: {last_node.key}"
        )
        assert len(kv_indices) == 7
        assert len(last_node.key) == 2

        mamba_num = 1
        print(f"evicting {mamba_num} mamba")
        result = tree.evict(EvictParams(num_tokens=0, mamba_num=mamba_num))
        assert (
            result.mamba_num_evicted >= mamba_num
        ), f"evicted {result.mamba_num_evicted} mamba states, expected {mamba_num}"
        tree.pretty_print()

        req8_token_ids = [1, 2, 3, 4, 5, 60, 70]
        result = tree.match_prefix(MatchPrefixParams(key=RadixKey(req8_token_ids)))
        kv_indices, last_node = result.device_indices, result.last_device_node
        print(
            f"req8: token_ids: {req8_token_ids}, matched kv_indices: {kv_indices}, last_node.key: {last_node.key}"
        )
        assert len(kv_indices) == 0
        assert len(last_node.key) == 0

        req9_token_ids = [1, 2, 3, 4, 5, 6, 7]
        req9 = make_dummy_req()
        result = tree.match_prefix(
            MatchPrefixParams(key=RadixKey(req9_token_ids), req=req9, cow_mamba=True)
        )
        kv_indices, last_node = result.device_indices, result.last_device_node
        assert req9.mamba_pool_idx is not None
        assert torch.all(
            mamba_pool.mamba_cache.conv[0][:, req9.mamba_pool_idx]
            == mamba_pool.mamba_cache.conv[0][:, last_node.mamba_value]
        )
        assert torch.all(
            mamba_pool.mamba_cache.temporal[:, req9.mamba_pool_idx]
            == mamba_pool.mamba_cache.temporal[:, last_node.mamba_value]
        )

        print(tree.available_and_evictable_str())
```
**EN:** This test exercises `test_mamba_radix_cache_1` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_1`。 这一段对应同一逻辑块的第 2 部分。

### Lines 306-307: test case mamba radix cache 1 (part 3/3) / 测试用例 mamba radix cache 1（第 3/3 部分）
```python
        print(available_and_evictable_str(tree))
        tree.sanity_check()
```
**EN:** This test exercises `test_mamba_radix_cache_1` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 3 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_1`。 这一段对应同一逻辑块的第 3 部分。

### Lines 309-361: test case mamba radix cache kv events / 测试用例 mamba radix cache kv events
```python
    def test_mamba_radix_cache_kv_events(self):
        tree, allocator, _, make_dummy_req = self._setup_tree_and_allocator(
            enable_kv_cache_events=True
        )
        tree.take_events()  # Clear the reset event.

        stored_hashes = []

        req1 = make_dummy_req()
        key1 = RadixKey([1, 2, 3])
        tree.insert(
            InsertParams(
                key=key1,
                value=allocator.alloc(3)[: len(key1)],
                mamba_value=req1.mamba_pool_idx.unsqueeze(0),
            )
        )
        events = tree.take_events()
        stored_events = [e for e in events if isinstance(e, BlockStored)]
        self.assertEqual(len(stored_events), 3)
        self.assertEqual([e.token_ids[0] for e in stored_events], [1, 2, 3])
        stored_hashes.extend(e.block_hashes[0] for e in stored_events)

        req2 = make_dummy_req()
        key2 = RadixKey([1, 2, 3, 4, 5])
        tree.insert(
            InsertParams(
                key=key2,
                value=allocator.alloc(5)[: len(key2)],
                mamba_value=req2.mamba_pool_idx.unsqueeze(0),
            )
        )
        events = tree.take_events()
        stored_events = [e for e in events if isinstance(e, BlockStored)]
        self.assertEqual(len(stored_events), 2)
        self.assertEqual([e.token_ids[0] for e in stored_events], [4, 5])
        stored_hashes.extend(e.block_hashes[0] for e in stored_events)

        # Evicting an internal mamba state creates a tombstone but does not
        # remove full-attention KV blocks, so it must not emit BlockRemoved.
        result = tree.evict(EvictParams(num_tokens=0, mamba_num=1))
        self.assertEqual(result.num_tokens_evicted, 0)
        self.assertEqual(result.mamba_num_evicted, 1)
        events = tree.take_events()
        self.assertEqual([e for e in events if isinstance(e, BlockRemoved)], [])

        result = tree.evict(EvictParams(num_tokens=1))
        self.assertGreaterEqual(result.num_tokens_evicted, 1)
        events = tree.take_events()
        removed_hashes = [
            e.block_hashes[0] for e in events if isinstance(e, BlockRemoved)
        ]
        self.assertCountEqual(removed_hashes, stored_hashes)
```
**EN:** This test exercises `test_mamba_radix_cache_kv_events` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_kv_events`。

### Lines 363-398: test case mamba radix cache kv events split hash / 测试用例 mamba radix cache kv events split hash
```python
    def test_mamba_radix_cache_kv_events_split_hash(self):
        tree, allocator, _, make_dummy_req = self._setup_tree_and_allocator(
            enable_kv_cache_events=True
        )
        tree.take_events()  # Clear the reset event.

        req1 = make_dummy_req()
        key1 = RadixKey([1, 2, 3, 4])
        tree.insert(
            InsertParams(
                key=key1,
                value=allocator.alloc(4)[: len(key1)],
                mamba_value=req1.mamba_pool_idx.unsqueeze(0),
            )
        )
        first_insert_events = [
            e for e in tree.take_events() if isinstance(e, BlockStored)
        ]
        self.assertEqual(len(first_insert_events), 4)
        split_parent_hash = first_insert_events[1].block_hashes[0]

        req2 = make_dummy_req()
        key2 = RadixKey([1, 2, 5, 6])
        tree.insert(
            InsertParams(
                key=key2,
                value=allocator.alloc(4)[: len(key2)],
                mamba_value=req2.mamba_pool_idx.unsqueeze(0),
            )
        )
        second_insert_events = [
            e for e in tree.take_events() if isinstance(e, BlockStored)
        ]
        self.assertEqual(len(second_insert_events), 2)
        self.assertEqual(second_insert_events[0].token_ids, [5])
        self.assertEqual(second_insert_events[0].parent_block_hash, split_parent_hash)
```
**EN:** This test exercises `test_mamba_radix_cache_kv_events_split_hash` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_kv_events_split_hash`。

### Lines 400-479: method setup tree and allocator (part 1/2) / 方法 setup tree and allocator（第 1/2 部分）
```python
    def _setup_tree_and_allocator(self, enable_kv_cache_events=False):
        """Helper to create a MambaRadixCache with allocator for testing."""
        set_global_server_args_for_scheduler(
            ServerArgs(model_path="dummy", page_size=1)
        )
        size = 128
        dtype = torch.bfloat16
        head_num = 2
        head_dim = 256
        num_layers = 48
        global_interval = 4
        max_num_reqs = 10
        mamba_cache_size = 20
        max_context_len = 128
        device = get_device()
        full_attention_layer_ids = [
            i for i in range(global_interval - 1, num_layers, global_interval)
        ]
        mamba_layers = [
            i for i in range(num_layers) if i not in full_attention_layer_ids
        ]
        with envs.SGLANG_MAMBA_SSM_DTYPE.override("bfloat16"):
            shape = Mamba2StateShape.create(
                tp_world_size=1,
                intermediate_size=4096,
                n_groups=16,
                num_heads=32,
                head_dim=128,
                state_size=128,
                conv_kernel=4,
            )
            mamba2_cache_params = Mamba2CacheParams(shape=shape, layers=mamba_layers)

        req_to_token_pool = HybridReqToTokenPool(
            size=max_num_reqs,
            mamba_size=mamba_cache_size,
            mamba_spec_state_size=max_num_reqs,
            max_context_len=max_context_len,
            device=device,
            enable_memory_saver=False,
            cache_params=mamba2_cache_params,
            mamba_layer_ids=mamba_layers,
            enable_mamba_extra_buffer=False,
            speculative_num_draft_tokens=3,
        )
        pool = HybridLinearKVPool(
            size=size,
            dtype=dtype,
            page_size=1,
            head_num=head_num,
            head_dim=head_dim,
            full_attention_layer_ids=full_attention_layer_ids,
            enable_kvcache_transpose=False,
            device=device,
            enable_memory_saver=False,
            mamba_pool=req_to_token_pool.mamba_pool,
        )
        allocator = TokenToKVPoolAllocator(
            size=size,
            dtype=dtype,
            device=device,
            kvcache=pool,
            need_sort=False,
        )
        params = CacheInitParams(
            req_to_token_pool=req_to_token_pool,
            token_to_kv_pool_allocator=allocator,
            page_size=1,
            disable=False,
            enable_kv_cache_events=enable_kv_cache_events,
        )
        tree = MambaRadixCache(params=params)

        def make_dummy_req():
            sampling_params = SamplingParams(
                temperature=0,
                max_new_tokens=1,
            )
            req = Req(
                rid=0,
```
**EN:** Helper to create a MambaRadixCache with allocator for testing. This block implements `_setup_tree_and_allocator` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** Helper to create a MambaRadixCache with allocator for testing. 该代码块实现 `_setup_tree_and_allocator`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 480-487: method setup tree and allocator (part 2/2) / 方法 setup tree and allocator（第 2/2 部分）
```python
                origin_input_text="",
                origin_input_ids=[],
                sampling_params=sampling_params,
            )
            req_to_token_pool.alloc([req])
            return req

        return tree, allocator, req_to_token_pool, make_dummy_req
```
**EN:** Helper to create a MambaRadixCache with allocator for testing. This block implements `_setup_tree_and_allocator` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** Helper to create a MambaRadixCache with allocator for testing. 该代码块实现 `_setup_tree_and_allocator`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 489-534: test case hi mamba tombstone cleanup respects host ref / 测试用例 hi mamba tombstone cleanup respects host ref
```python
    def test_hi_mamba_tombstone_cleanup_respects_host_ref(self):
        tree = object.__new__(HiMambaRadixCache)
        root = TreeNode()
        parent = TreeNode()
        deleted = TreeNode()

        root.key = RadixKey([])
        parent.key = RadixKey([1])
        deleted.key = RadixKey([2])
        parent.parent = root
        deleted.parent = parent
        parent.value = torch.tensor([1], dtype=torch.int64)
        parent.protect_host()
        root.children[parent.key.child_key(1)] = parent

        class RecordingCacheController:
            def __init__(self):
                self.device_evictions = []
                self.host_evictions = []

            def evict_device(self, value):
                self.device_evictions.append(value)

            def evict_host(self, value):
                self.host_evictions.append(value)

        tree.root_node = root
        tree.page_size = 1
        tree.full_lru_list = LRUList(mamba=False)
        tree.full_lru_list.insert_mru(parent)
        tree.cache_controller = RecordingCacheController()
        tree.full_evictable_size_ = len(parent.value)
        tree.evictable_full_device_leaves = {parent}
        tree.evictable_full_host_leaves = set()

        result_node, full_evicted, mamba_evicted = (
            tree._iteratively_delete_tombstone_leaf(deleted)
        )

        self.assertIs(result_node, deleted)
        self.assertEqual(full_evicted, 0)
        self.assertEqual(mamba_evicted, 0)
        self.assertIs(root.children[parent.key.child_key(1)], parent)
        self.assertTrue(tree.full_lru_list.in_list(parent))
        self.assertEqual(tree.cache_controller.device_evictions, [])
        self.assertEqual(tree.cache_controller.host_evictions, [])
```
**EN:** This test exercises `test_hi_mamba_tombstone_cleanup_respects_host_ref` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hi_mamba_tombstone_cleanup_respects_host_ref`。

### Lines 536-582: test case mamba pool cpu offload / 测试用例 mamba pool cpu offload
```python
    def test_mamba_pool_cpu_offload(self):
        """MambaPool.get_cpu_copy / load_cpu_copy round-trips conv and temporal state."""
        _, _, req_to_token_pool, _ = self._setup_tree_and_allocator()
        mamba_pool = req_to_token_pool.mamba_pool
        n = 3
        indices = mamba_pool.alloc(n)
        self.assertIsNotNone(indices)

        # Write known sentinel values at the allocated slots.
        for conv in mamba_pool.mamba_cache.conv:
            conv[:, indices] = 1.0
        mamba_pool.mamba_cache.temporal[:, indices] = 2.0

        # Save to CPU.
        conv_cpu, temporal_cpu = mamba_pool.get_cpu_copy(indices)

        # Verify CPU tensors match what was written.
        for i, conv in enumerate(mamba_pool.mamba_cache.conv):
            expected = conv[:, indices].cpu()
            self.assertTrue(
                torch.allclose(conv_cpu[i].float(), expected.float()),
                f"conv[{i}] CPU copy mismatch",
            )
        expected_t = mamba_pool.mamba_cache.temporal[:, indices].cpu()
        self.assertTrue(
            torch.allclose(temporal_cpu.float(), expected_t.float()),
            "temporal CPU copy mismatch",
        )

        # Zero out GPU slots and restore from CPU copy.
        for conv in mamba_pool.mamba_cache.conv:
            conv[:, indices] = 0.0
        mamba_pool.mamba_cache.temporal[:, indices] = 0.0

        mamba_pool.load_cpu_copy((conv_cpu, temporal_cpu), indices)

        # Verify restored values match the sentinels.
        for conv in mamba_pool.mamba_cache.conv:
            restored = conv[:, indices]
            self.assertTrue(
                torch.all(restored == 1.0),
                "conv not restored after load_cpu_copy",
            )
        self.assertTrue(
            torch.all(mamba_pool.mamba_cache.temporal[:, indices] == 2.0),
            "temporal not restored after load_cpu_copy",
        )
```
**EN:** MambaPool.get_cpu_copy / load_cpu_copy round-trips conv and temporal state. This test exercises `test_mamba_pool_cpu_offload` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** MambaPool.get_cpu_copy / load_cpu_copy round-trips conv and temporal state. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_pool_cpu_offload`。

### Lines 584-657: test case hybrid kv pool cpu offload / 测试用例 hybrid kv pool cpu offload
```python
    def test_hybrid_kv_pool_cpu_offload(self):
        """HybridLinearKVPool.get_cpu_copy / load_cpu_copy saves and restores both
        the full-attention KV cache and Mamba state in a single round-trip."""
        _, allocator, req_to_token_pool, _ = self._setup_tree_and_allocator()
        mamba_pool = req_to_token_pool.mamba_pool
        hybrid_pool = allocator._kvcache  # HybridLinearKVPool

        self.assertIsInstance(hybrid_pool, HybridLinearKVPool)

        n_tokens = 4
        kv_indices = allocator.alloc(n_tokens)
        self.assertIsNotNone(kv_indices)
        mamba_indices = mamba_pool.alloc(1)
        self.assertIsNotNone(mamba_indices)

        # Write sentinel values into KV buffers (all full-attention layers).
        for layer_id in range(hybrid_pool.full_kv_pool.layer_num):
            hybrid_pool.full_kv_pool.k_buffer[layer_id][kv_indices] = 3.0
            hybrid_pool.full_kv_pool.v_buffer[layer_id][kv_indices] = 4.0

        # Write sentinel values into Mamba state.
        for conv in mamba_pool.mamba_cache.conv:
            conv[:, mamba_indices] = 5.0
        mamba_pool.mamba_cache.temporal[:, mamba_indices] = 6.0

        # --- Round-trip with Mamba indices provided ---
        cpu_copy = allocator.get_cpu_copy(kv_indices, mamba_indices=mamba_indices)
        kv_cpu, mamba_cpu = cpu_copy
        self.assertIsNotNone(
            mamba_cpu, "mamba_cpu should be saved when mamba_indices given"
        )

        # Zero out GPU.
        for layer_id in range(hybrid_pool.full_kv_pool.layer_num):
            hybrid_pool.full_kv_pool.k_buffer[layer_id][kv_indices] = 0.0
            hybrid_pool.full_kv_pool.v_buffer[layer_id][kv_indices] = 0.0
        for conv in mamba_pool.mamba_cache.conv:
            conv[:, mamba_indices] = 0.0
        mamba_pool.mamba_cache.temporal[:, mamba_indices] = 0.0

        allocator.load_cpu_copy(cpu_copy, kv_indices, mamba_indices=mamba_indices)

        # Verify KV restored.
        for layer_id in range(hybrid_pool.full_kv_pool.layer_num):
            self.assertTrue(
                torch.all(
                    hybrid_pool.full_kv_pool.k_buffer[layer_id][kv_indices] == 3.0
                ),
                f"k_buffer layer {layer_id} not restored",
            )
            self.assertTrue(
                torch.all(
                    hybrid_pool.full_kv_pool.v_buffer[layer_id][kv_indices] == 4.0
                ),
                f"v_buffer layer {layer_id} not restored",
            )

        # Verify Mamba restored.
        for conv in mamba_pool.mamba_cache.conv:
            self.assertTrue(
                torch.all(conv[:, mamba_indices] == 5.0),
                "conv not restored after load_cpu_copy",
            )
        self.assertTrue(
            torch.all(mamba_pool.mamba_cache.temporal[:, mamba_indices] == 6.0),
            "temporal not restored after load_cpu_copy",
        )

        # --- Without mamba_indices: mamba_cpu must be None ---
        cpu_copy_no_mamba = allocator.get_cpu_copy(kv_indices, mamba_indices=None)
        _, mamba_cpu_none = cpu_copy_no_mamba
        self.assertIsNone(
            mamba_cpu_none, "mamba_cpu should be None when mamba_indices=None"
        )
```
**EN:** HybridLinearKVPool.get_cpu_copy / load_cpu_copy saves and restores both the full-attention KV cache and Mamba state in a single round-trip. This test exercises `test_hybrid_kv_pool_cpu_offload` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** HybridLinearKVPool.get_cpu_copy / load_cpu_copy saves and restores both the full-attention KV cache and Mamba state in a single round-trip. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hybrid_kv_pool_cpu_offload`。

### Lines 659-732: test case insert prev prefix len / 测试用例 insert prev prefix len
```python
    def test_insert_prev_prefix_len(self):
        """Test that prev_prefix_len correctly controls which KV indices are freed
        during insert, covering: full free, partial free across multi-node, and no free.
        """
        tree, allocator, req_to_token_pool, make_dummy_req = (
            self._setup_tree_and_allocator()
        )

        initial_avail = allocator.available_size()

        # Step 1: Insert [1,2,3] to create first node
        req1 = make_dummy_req()
        key1 = RadixKey([1, 2, 3])
        tree.insert(
            InsertParams(
                key=key1,
                value=allocator.alloc(3)[: len(key1)],
                mamba_value=req1.mamba_pool_idx.unsqueeze(0),
            )
        )
        assert allocator.available_size() == initial_avail - 3

        # Step 2: Insert [1,2,3,4,5,6,7] with prev_prefix_len=0 (free all matched)
        # Creates tree: [1,2,3] -> [4,5,6,7]
        req2 = make_dummy_req()
        key2 = RadixKey([1, 2, 3, 4, 5, 6, 7])
        result = tree.insert(
            InsertParams(
                key=key2,
                value=allocator.alloc(7)[: len(key2)],
                mamba_value=req2.mamba_pool_idx.unsqueeze(0),
                prev_prefix_len=0,
            )
        )
        assert result.prefix_len == 3
        # alloc 7, freed 3 (dup prefix [0..2]), stored 4 in new node => net -4
        assert allocator.available_size() == initial_avail - 3 - 4
        avail_after_step2 = allocator.available_size()

        # Step 3: Insert [1,2,3,4,5,6,7,8] with prev_prefix_len=2
        # Matched prefix = 7 (across two nodes: [1,2,3] len=3, [4,5,6,7] len=4)
        # Protected [0..1], freed [2..6] = 5 slots, new [7] = 1 slot stored
        req3 = make_dummy_req()
        key3 = RadixKey([1, 2, 3, 4, 5, 6, 7, 8])
        result = tree.insert(
            InsertParams(
                key=key3,
                value=allocator.alloc(8)[: len(key3)],
                mamba_value=req3.mamba_pool_idx.unsqueeze(0),
                prev_prefix_len=2,
            )
        )
        assert result.prefix_len == 7
        # alloc 8, freed 5, stored 1 => net -3
        assert allocator.available_size() == avail_after_step2 - 3
        avail_after_step3 = allocator.available_size()

        # Step 4: Insert [1,2,3,4,5,6,7,8,9] with prev_prefix_len=8 (covers all matched)
        # Matched prefix = 8, prev_prefix_len=8 => nothing freed
        req4 = make_dummy_req()
        key4 = RadixKey([1, 2, 3, 4, 5, 6, 7, 8, 9])
        result = tree.insert(
            InsertParams(
                key=key4,
                value=allocator.alloc(9)[: len(key4)],
                mamba_value=req4.mamba_pool_idx.unsqueeze(0),
                prev_prefix_len=8,
            )
        )
        assert result.prefix_len == 8
        # alloc 9, freed 0, stored 1 => net -9
        assert allocator.available_size() == avail_after_step3 - 9

        tree.sanity_check()
```
**EN:** Test that prev_prefix_len correctly controls which KV indices are freed during insert, covering: full free, partial free across multi-node, and no free. This test exercises `test_insert_prev_prefix_len` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that prev_prefix_len correctly controls which KV indices are freed during insert, covering: full free, partial free across multi-node, and no free. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_insert_prev_prefix_len`。

### Lines 735-736: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMamba`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMamba.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMamba.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestMamba.test_hybrid_linear_kv_pool`: This test exercises `test_hybrid_linear_kv_pool` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hybrid_linear_kv_pool`。
- `TestMamba.test_mamba_pool`: This test exercises `test_mamba_pool` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_pool`。
- `TestMamba.test_mamba_radix_cache_1`: This test exercises `test_mamba_radix_cache_1` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_1`。
- `TestMamba.test_mamba_radix_cache_kv_events`: This test exercises `test_mamba_radix_cache_kv_events` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_kv_events`。
- `TestMamba.test_mamba_radix_cache_kv_events_split_hash`: This test exercises `test_mamba_radix_cache_kv_events_split_hash` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_radix_cache_kv_events_split_hash`。
- `TestMamba._setup_tree_and_allocator`: Helper to create a MambaRadixCache with allocator for testing. / 该代码块实现 `_setup_tree_and_allocator`，承担模块行为中的一个聚焦逻辑片段。
- `TestMamba.test_hi_mamba_tombstone_cleanup_respects_host_ref`: This test exercises `test_hi_mamba_tombstone_cleanup_respects_host_ref` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hi_mamba_tombstone_cleanup_respects_host_ref`。
- `TestMamba.test_mamba_pool_cpu_offload`: MambaPool.get_cpu_copy / load_cpu_copy round-trips conv and temporal state. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mamba_pool_cpu_offload`。
- `TestMamba.test_hybrid_kv_pool_cpu_offload`: HybridLinearKVPool.get_cpu_copy / load_cpu_copy saves and restores both the full-attention KV cache and Mamba state in a single round-trip. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hybrid_kv_pool_cpu_offload`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.disaggregation.kv_events`, `sglang.srt.environ`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.common`, `sglang.srt.mem_cache.hi_mamba_radix_cache`, `sglang.srt.mem_cache.mamba_radix_cache`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.sampling.sampling_params`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 736
