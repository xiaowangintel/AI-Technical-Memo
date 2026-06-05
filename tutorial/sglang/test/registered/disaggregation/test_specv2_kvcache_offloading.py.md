# test_specv2_kvcache_offloading.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/disaggregation/test_specv2_kvcache_offloading.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on disaggregation specv2 kvcache offloading in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 disaggregation specv2 kvcache offloading 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Document the module
```python
"""
Unit tests for _release_finished_req in DecodeKVCacheOffloadManager.

Verifies that over-allocated KV cache slots (from speculative decoding v2)
are correctly freed when a request finishes, preventing GPU memory leaks.

Requires: torch, sglang (run in an environment with sglang installed)
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 10-18: Import dependencies
```python
import unittest
from unittest.mock import MagicMock

import torch

from sglang.srt.disaggregation.decode_kvcache_offload_manager import (
    DecodeKVCacheOffloadManager,
)
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 20-20: Register CI metadata
```python
register_cuda_ci(est_time=8, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 23-52: Define helper: make mock req
```python
def _make_mock_req(
    req_pool_idx: int,
    kv_committed_len: int,
    kv_allocated_len: int,
    prefix_indices_len: int = 0,
    rid: int = 0,
):
    """Create a mock Req with the KV cache state needed for testing."""
    req = MagicMock()
    req.rid = rid
    req.req_pool_idx = req_pool_idx
    req.kv_committed_len = kv_committed_len
    req.kv_allocated_len = kv_allocated_len
    req.kv_committed_freed = False
    req.kv_overallocated_freed = False
    req.prefix_indices = list(range(prefix_indices_len))

    def pop_committed():
        assert not req.kv_committed_freed
        req.kv_committed_freed = True
        return req.kv_committed_len

    def pop_overallocated():
        assert not req.kv_overallocated_freed
        req.kv_overallocated_freed = True
        return req.kv_committed_len, req.kv_allocated_len

    req.pop_committed_kv_cache = pop_committed
    req.pop_overallocated_kv_cache = pop_overallocated
    return req
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 55-81: Define helper: make manager
```python
def _make_manager(pool_size: int, page_size: int = 1):
    """Create a DecodeKVCacheOffloadManager with mock pools for testing."""
    # Build a real req_to_token tensor so indexing works
    req_to_token = torch.arange(pool_size, dtype=torch.int64).unsqueeze(0)

    req_to_token_pool = MagicMock()
    req_to_token_pool.req_to_token = req_to_token

    freed_indices = []

    allocator = MagicMock()
    allocator.free = MagicMock(
        side_effect=lambda idx: freed_indices.append(idx.clone())
    )

    tree_cache = MagicMock()
    tree_cache.protected_size_ = 0

    # Bypass __init__ entirely and set attributes directly
    manager = object.__new__(DecodeKVCacheOffloadManager)
    manager.req_to_token_pool = req_to_token_pool
    manager.token_to_kv_pool_allocator = allocator
    manager.page_size = page_size
    manager.tree_cache = tree_cache
    manager.offloaded_state = {}

    return manager, freed_indices
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 84-84: Define class TestReleaseFinishedReq
```python
class TestReleaseFinishedReq(unittest.TestCase):
```
**EN:** This declaration introduces the `TestReleaseFinishedReq` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReleaseFinishedReq` 测试类，并说明它通过继承承担的职责。

### Lines 85-85: Document the class `TestReleaseFinishedReq`
```python
    """Tests for _release_finished_req overallocation cleanup."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestReleaseFinishedReq`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestReleaseFinishedReq`的设计意图。

### Lines 87-103: Run test: no overallocation
```python
    def test_no_overallocation(self):
        """Without spec v2, kv_committed == kv_allocated; no extra free."""
        manager, freed = _make_manager(pool_size=32)
        req = _make_mock_req(
            req_pool_idx=0,
            kv_committed_len=20,
            kv_allocated_len=20,  # no overallocation
        )
        prefill_offloaded_len = 8

        manager._release_finished_req(req, prefill_offloaded_len)

        # Only one free call: the committed range [8:20]
        self.assertEqual(len(freed), 1)
        expected = torch.arange(8, 20, dtype=torch.int64)
        self.assertTrue(torch.equal(freed[0], expected))
        manager.req_to_token_pool.free.assert_called_once_with(req)
```
**EN:** This test method exercises no overallocation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no overallocation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 105-123: Run test: with overallocation
```python
    def test_with_overallocation(self):
        """With spec v2, overallocated slots [committed:allocated] must be freed."""
        manager, freed = _make_manager(pool_size=32)
        req = _make_mock_req(
            req_pool_idx=0,
            kv_committed_len=20,
            kv_allocated_len=28,  # 8 over-allocated slots
        )
        prefill_offloaded_len = 8

        manager._release_finished_req(req, prefill_offloaded_len)

        # Two free calls: committed [8:20] and overallocated [20:28]
        self.assertEqual(len(freed), 2)
        expected_committed = torch.arange(8, 20, dtype=torch.int64)
        expected_overalloc = torch.arange(20, 28, dtype=torch.int64)
        self.assertTrue(torch.equal(freed[0], expected_committed))
        self.assertTrue(torch.equal(freed[1], expected_overalloc))
        manager.req_to_token_pool.free.assert_called_once_with(req)
```
**EN:** This test method exercises with overallocation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with overallocation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 125-144: Run test: overallocation with page alignment
```python
    def test_overallocation_with_page_alignment(self):
        """With page_size > 1, start of overallocated range is ceil-aligned."""
        page_size = 4
        manager, freed = _make_manager(pool_size=32, page_size=page_size)
        req = _make_mock_req(
            req_pool_idx=0,
            kv_committed_len=10,  # not page-aligned
            kv_allocated_len=28,
        )
        prefill_offloaded_len = 4

        manager._release_finished_req(req, prefill_offloaded_len)

        # Committed range [4:10]
        # Overallocated: start_p = ceil_align(10, 4) = 12, end_p = 28 => [12:28]
        self.assertEqual(len(freed), 2)
        expected_committed = torch.arange(4, 10, dtype=torch.int64)
        expected_overalloc = torch.arange(12, 28, dtype=torch.int64)
        self.assertTrue(torch.equal(freed[0], expected_committed))
        self.assertTrue(torch.equal(freed[1], expected_overalloc))
```
**EN:** This test method exercises overallocation with page alignment and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 overallocation with page alignment 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 146-162: Run test: overallocation page aligned noop
```python
    def test_overallocation_page_aligned_noop(self):
        """When ceil_align(committed, page_size) >= allocated, no overalloc free."""
        page_size = 4
        manager, freed = _make_manager(pool_size=32, page_size=page_size)
        req = _make_mock_req(
            req_pool_idx=0,
            kv_committed_len=10,  # ceil_align(10, 4) = 12
            kv_allocated_len=12,  # same as aligned start
        )
        prefill_offloaded_len = 4

        manager._release_finished_req(req, prefill_offloaded_len)

        # Only committed [4:10], no overalloc because start_p == end_p
        self.assertEqual(len(freed), 1)
        expected_committed = torch.arange(4, 10, dtype=torch.int64)
        self.assertTrue(torch.equal(freed[0], expected_committed))
```
**EN:** This test method exercises overallocation page aligned noop and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 overallocation page aligned noop 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 164-177: Run test: prefix indices decremented
```python
    def test_prefix_indices_decremented(self):
        """protected_size_ is decremented by len(req.prefix_indices)."""
        manager, _ = _make_manager(pool_size=32)
        manager.tree_cache.protected_size_ = 10
        req = _make_mock_req(
            req_pool_idx=0,
            kv_committed_len=20,
            kv_allocated_len=20,
            prefix_indices_len=5,
        )

        manager._release_finished_req(req, start_offset=0)

        self.assertEqual(manager.tree_cache.protected_size_, 5)
```
**EN:** This test method exercises prefix indices decremented and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prefix indices decremented 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 180-181: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.disaggregation.decode_kvcache_offload_manager`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`, `unittest.mock`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
