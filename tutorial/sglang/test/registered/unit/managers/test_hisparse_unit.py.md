# test_hisparse_unit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_hisparse_unit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hisparse unit behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 hisparse unit 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: supporting statements / 辅助语句
```python
"""Unit tests for HiSparse hierarchical sparse KV cache system.

Tests cover:
- CUDA kernel correctness (swap_in_selected_pages vs naive_load_topk oracle)
- Memory allocator lifecycle (alloc / free / available_size)
- Request lifecycle (staging path, direct-to-host path)
- Batch multi-request correctness
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 10-17: module imports and dependencies / 模块导入与依赖
```python
import os
import unittest
from types import SimpleNamespace

import torch

from sglang.srt.utils import is_cuda, is_hip, is_npu, is_xpu
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `unittest`, `types`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `unittest`, `types`, `torch`。

### Lines 19-34: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=10, stage="base-b", runner_config="1-gpu-small")

# ---------------------------------------------------------------------------
# Test configuration (small-scale for fast CI runs)
# ---------------------------------------------------------------------------
SIZE = 2048  # device buffer pool size (tokens)
PAGE_SIZE = 64  # page size (must be 64 for CUDA, 1 for ROCm)
TOP_K = 256  # top-k selection count
DEVICE_BUFFER_SIZE = 512  # device buffer per request
HOST_TO_DEVICE_RATIO = 2
KV_LORA_RANK = 512
QK_ROPE_HEAD_DIM = 64
KV_CACHE_DIM = 576  # MLA dim (DeepSeek-style)
LAYER_NUM = 2
MAX_NUM_REQS = 8
MAX_CONTEXT_LEN = 2048
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 37-58: function make req / 函数 make req
```python
def _make_req(rid="test-req-0", origin_input_ids=None, output_ids=None):
    """Create a minimal mock Req object with the fields HiSparseCoordinator uses."""
    if origin_input_ids is None:
        origin_input_ids = list(range(64))
    if output_ids is None:
        output_ids = []
    req = SimpleNamespace(
        rid=rid,
        origin_input_ids=origin_input_ids,
        output_ids=output_ids,
        fill_ids=origin_input_ids + output_ids,
        seqlen=len(origin_input_ids) + len(output_ids),
        req_pool_idx=None,
        kv_allocated_len=0,
        kv_committed_len=0,
        finished_reason=None,
        hisparse_staging=False,
        staging=False,
        is_chunked=0,
    )
    req.finished = lambda: req.finished_reason is not None
    return req
```
**EN:** Create a minimal mock Req object with the fields HiSparseCoordinator uses. This block implements `_make_req` and captures one focused piece of the module's behavior.
**CN:** Create a minimal mock Req object with the fields HiSparseCoordinator uses. 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 61-61: class TestHiSparseUnit declaration / 类 TestHiSparseUnit 声明
```python
class TestHiSparseUnit(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 62-62: supporting statements / 辅助语句
```python
    """Test class that builds a minimal HiSparse component stack."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 63-67: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Fixture
    # ==================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 68-141: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is required for HiSparse tests.")
        if is_npu() or is_xpu():
            raise unittest.SkipTest("HiSparse tests only support CUDA/ROCm.")
        if not (is_cuda() or is_hip()):
            raise unittest.SkipTest("CUDA/ROCm not available.")

        os.environ.setdefault("MASTER_ADDR", "127.0.0.1")
        os.environ.setdefault("MASTER_PORT", "29599")
        if not torch.distributed.is_initialized():
            torch.distributed.init_process_group(backend="gloo", rank=0, world_size=1)
        cls.tp_group = torch.distributed.group.WORLD

        from sglang.srt.mem_cache.memory_pool_host import (
            ALLOC_MEMORY_FUNCS,
            alloc_with_pin_memory,
        )

        cls._original_alloc = ALLOC_MEMORY_FUNCS["cuda"]
        ALLOC_MEMORY_FUNCS["cuda"] = alloc_with_pin_memory

        global_page_size = 1 if is_hip() else PAGE_SIZE

        from sglang.srt.mem_cache.hisparse_memory_pool import (
            HiSparseNSATokenToKVPool,
            HiSparseTokenToKVPoolAllocator,
        )

        cls.device_pool = HiSparseNSATokenToKVPool(
            size=SIZE,
            page_size=global_page_size,
            kv_lora_rank=KV_LORA_RANK,
            dtype=torch.bfloat16,
            qk_rope_head_dim=QK_ROPE_HEAD_DIM,
            layer_num=LAYER_NUM,
            device="cuda",
            index_head_dim=128,
            enable_memory_saver=False,
            kv_cache_dim=KV_CACHE_DIM,
            host_to_device_ratio=HOST_TO_DEVICE_RATIO,
        )
        cls.allocator = HiSparseTokenToKVPoolAllocator(
            size=SIZE,
            page_size=global_page_size,
            dtype=torch.bfloat16,
            device="cuda",
            kvcache=cls.device_pool,
            need_sort=False,
            host_to_device_ratio=HOST_TO_DEVICE_RATIO,
        )

        from sglang.srt.mem_cache.memory_pool import ReqToTokenPool

        cls.req_to_token_pool = ReqToTokenPool(
            size=MAX_NUM_REQS,
            max_context_len=MAX_CONTEXT_LEN,
            device="cuda",
            enable_memory_saver=False,
        )

        from sglang.srt.managers.hisparse_coordinator import HiSparseCoordinator

        cls.page_size = global_page_size
        cls.coordinator = HiSparseCoordinator(
            req_to_token_pool=cls.req_to_token_pool,
            token_to_kv_pool_allocator=cls.allocator,
            top_k=TOP_K,
            device_buffer_size=DEVICE_BUFFER_SIZE,
            device="cuda",
            tp_group=cls.tp_group,
            host_to_device_ratio=HOST_TO_DEVICE_RATIO,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 143-149: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        from sglang.srt.mem_cache.memory_pool_host import ALLOC_MEMORY_FUNCS

        ALLOC_MEMORY_FUNCS["cuda"] = cls._original_alloc
        if torch.distributed.is_initialized():
            torch.distributed.destroy_process_group()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 151-170: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Reset shared allocator / coordinator state so tests are isolated.

        Without this, a mid-test assertion failure skips cleanup and leaks
        resources, causing unrelated failures in later tests.
        """
        self.allocator.clear()
        self.req_to_token_pool.clear()
        self.coordinator.mem_pool_host.clear()
        # Reset per-request coordinator bookkeeping
        self.coordinator.req_to_device_buffer.zero_()
        self.coordinator.req_device_buffer_size.zero_()
        self.coordinator.req_to_host_pool.fill_(-1)
        self.coordinator.req_device_buffer_tokens.fill_(-1)
        self.coordinator.req_device_buffer_token_locs.fill_(-1)
        self.coordinator.lru_slots[:] = self.coordinator._lru_init.view(1, 1, -1)
        self.coordinator.ack_staging_queue.clear()
        self.coordinator._has_pending_backup = False
        for i in range(len(self.coordinator._skip_first_backup)):
            self.coordinator._skip_first_backup[i] = False
```
**EN:** Reset shared allocator / coordinator state so tests are isolated. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Reset shared allocator / coordinator state so tests are isolated. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 171-175: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Low-level helpers
    # ==================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 176-180: method alloc req slot / 方法 alloc req slot
```python
    def _alloc_req_slot(self, req):
        """Allocate a req_pool_idx for the request."""
        indices = self.req_to_token_pool.alloc([req])
        self.assertIsNotNone(indices, "Failed to allocate req pool slot")
        return req.req_pool_idx
```
**EN:** Allocate a req_pool_idx for the request. This block implements `_alloc_req_slot` and captures one focused piece of the module's behavior.
**CN:** Allocate a req_pool_idx for the request. 该代码块实现 `_alloc_req_slot`，承担模块行为中的一个聚焦逻辑片段。

### Lines 182-185: method free req slot / 方法 free req slot
```python
    def _free_req_slot(self, req):
        """Free the req_pool_idx."""
        if req.req_pool_idx is not None:
            self.req_to_token_pool.free(req)
```
**EN:** Free the req_pool_idx. This block implements `_free_req_slot` and captures one focused piece of the module's behavior.
**CN:** Free the req_pool_idx. 该代码块实现 `_free_req_slot`，承担模块行为中的一个聚焦逻辑片段。

### Lines 187-210: method alloc kv / 方法 alloc kv
```python
    def _alloc_kv(self, req, fill_len, *, logical_only=False):
        """Allocate KV indices, write req_to_token_pool, update req fields.
        If logical_only=True, uses alloc_logical_only (PD-separated path).
        Returns kv_loc tensor."""
        device = self.allocator.device
        alloc_fn = (
            self.allocator.alloc_logical_only
            if logical_only
            else self.allocator.alloc_extend
        )
        kv_loc = alloc_fn(
            prefix_lens=torch.tensor([0], dtype=torch.int64, device=device),
            prefix_lens_cpu=torch.tensor([0], dtype=torch.int64),
            seq_lens=torch.tensor([fill_len], dtype=torch.int64, device=device),
            seq_lens_cpu=torch.tensor([fill_len], dtype=torch.int64),
            last_loc=torch.tensor([-1], dtype=torch.int64, device=device),
            extend_num_tokens=fill_len,
        )
        self.assertIsNotNone(kv_loc, "KV alloc failed")
        self.req_to_token_pool.write((req.req_pool_idx, slice(0, len(kv_loc))), kv_loc)
        req.kv_allocated_len = fill_len
        req.kv_committed_len = fill_len
        req.fill_ids = list(range(fill_len))
        return kv_loc
```
**EN:** Allocate KV indices, write req_to_token_pool, update req fields. This block implements `_alloc_kv` and captures one focused piece of the module's behavior.
**CN:** Allocate KV indices, write req_to_token_pool, update req fields. 该代码块实现 `_alloc_kv`，承担模块行为中的一个聚焦逻辑片段。

### Lines 211-215: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Mid-level helpers
    # ==================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 216-220: method kv pattern / 方法 kv pattern
```python
    @staticmethod
    def _kv_pattern(layer_id, token_id):
        """Deterministic KV value for (layer, token) — used by write & verify."""
        v = (layer_id * 10000 + token_id + 1) * 0.001
        return float(torch.tensor(v, dtype=torch.bfloat16))
```
**EN:** Deterministic KV value for (layer, token) — used by write & verify. This block implements `_kv_pattern` and captures one focused piece of the module's behavior.
**CN:** Deterministic KV value for (layer, token) — used by write & verify. 该代码块实现 `_kv_pattern`，承担模块行为中的一个聚焦逻辑片段。

### Lines 222-234: method write device patterns / 方法 write device patterns
```python
    def _write_device_patterns(self, kv_loc, fill_len):
        """Write distinguishable patterns into device KV buffer for all layers.

        kv_loc contains *logical* indices; we must translate them to hisparse
        device indices before indexing kv_buffer (which is sized for the
        hisparse pool, not the larger logical space).
        """
        hisparse_locs = self.allocator.full_to_hisparse_device_index_mapping[kv_loc]
        for lid in range(LAYER_NUM):
            for i in range(fill_len):
                self.device_pool.kv_buffer[lid][hisparse_locs[i]] = self._kv_pattern(
                    lid, i
                )
```
**EN:** Write distinguishable patterns into device KV buffer for all layers. This block implements `_write_device_patterns` and captures one focused piece of the module's behavior.
**CN:** Write distinguishable patterns into device KV buffer for all layers. 该代码块实现 `_write_device_patterns`，承担模块行为中的一个聚焦逻辑片段。

### Lines 236-247: method populate host pool / 方法 populate host pool
```python
    def _populate_host_pool(self, req, fill_len):
        """Allocate host slots, write known patterns, register in coordinator.
        Returns host_indices (cuda tensor)."""
        host_pool = self.coordinator.mem_pool_host
        host_indices = host_pool.alloc(fill_len)
        self.assertIsNotNone(host_indices, "Host alloc failed")
        host_indices = host_indices.to(device="cuda")
        self.coordinator.req_to_host_pool[req.req_pool_idx, :fill_len] = host_indices
        for lid in range(LAYER_NUM):
            for i in range(fill_len):
                host_pool.kv_buffer[lid][host_indices[i]] = self._kv_pattern(lid, i)
        return host_indices
```
**EN:** Allocate host slots, write known patterns, register in coordinator. This block implements `_populate_host_pool` and captures one focused piece of the module's behavior.
**CN:** Allocate host slots, write known patterns, register in coordinator. 该代码块实现 `_populate_host_pool`，承担模块行为中的一个聚焦逻辑片段。

### Lines 249-274: method build topk tokens / 方法 build topk tokens
```python
    def _build_topk_tokens(self, fill_len, *, include_newest=False):
        """Build a 1-D [TOP_K] int32 cuda tensor of token positions.

        If include_newest=True, fill_len-1 is guaranteed as the last valid slot.
        Pads with -1 when fill_len (or fill_len-1) < TOP_K.

        For long-sequence tests (fill_len > DEVICE_BUFFER_SIZE) where the
        "newest token" reserved slot is not populated (it requires an actual
        decode step + map_last_loc_to_buffer), callers should pass
        ``fill_len - 1`` as the effective pool size so position fill_len-1 is
        never randomly selected.
        """
        n = min(fill_len, TOP_K)
        if include_newest and n > 1:
            tokens = torch.randperm(fill_len - 1, device="cuda")[: n - 1].to(
                torch.int32
            )
            tokens = torch.cat(
                [tokens, torch.tensor([fill_len - 1], dtype=torch.int32, device="cuda")]
            )
        else:
            tokens = torch.randperm(fill_len, device="cuda")[:n].to(torch.int32)
        if n < TOP_K:
            pad = torch.full((TOP_K - n,), -1, dtype=torch.int32, device="cuda")
            tokens = torch.cat([tokens, pad])
        return tokens
```
**EN:** Build a 1-D [TOP_K] int32 cuda tensor of token positions. This block implements `_build_topk_tokens` and captures one focused piece of the module's behavior.
**CN:** Build a 1-D [TOP_K] int32 cuda tensor of token positions. 该代码块实现 `_build_topk_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 276-282: method make batch tensors / 方法 make batch tensors
```python
    def _make_batch_tensors(self, reqs, fill_lens):
        """Build (req_pool_indices [int64], seq_lens [int32]) on cuda."""
        rpi = torch.tensor(
            [r.req_pool_idx for r in reqs], dtype=torch.int64, device="cuda"
        )
        sls = torch.tensor(fill_lens, dtype=torch.int32, device="cuda")
        return rpi, sls
```
**EN:** Build (req_pool_indices [int64], seq_lens [int32]) on cuda. This block implements `_make_batch_tensors` and captures one focused piece of the module's behavior.
**CN:** Build (req_pool_indices [int64], seq_lens [int32]) on cuda. 该代码块实现 `_make_batch_tensors`，承担模块行为中的一个聚焦逻辑片段。

### Lines 284-300: method assert kv correct / 方法 assert kv correct
```python
    def _assert_kv_correct(self, locs_row, tokens_row, layer_id, count, msg=""):
        """Assert device KV data at *locs_row[:count]* matches the written
        pattern for the corresponding *tokens_row[:count]* positions."""
        for i in range(count):
            tok = int(tokens_row[i].item())
            if tok < 0:
                continue
            expected = self._kv_pattern(layer_id, tok)
            actual = self.device_pool.kv_buffer[layer_id][locs_row[i].long()]
            self.assertTrue(
                torch.allclose(
                    actual.float(),
                    torch.full_like(actual.float(), expected),
                    atol=1e-2,
                ),
                f"{msg}layer {layer_id}, token {tok}: KV data mismatch",
            )
```
**EN:** Assert device KV data at *locs_row[:count]* matches the written pattern for the corresponding *tokens_row[:count]* positions. This block implements `_assert_kv_correct` and captures one focused piece of the module's behavior.
**CN:** Assert device KV data at *locs_row[:count]* matches the written pattern for the corresponding *tokens_row[:count]* positions. 该代码块实现 `_assert_kv_correct`，承担模块行为中的一个聚焦逻辑片段。

### Lines 302-318: method assert matches naive / 方法 assert matches naive
```python
    def _assert_matches_naive(self, rpi, sls, batch, kernel_locs, layer_id, msg=""):
        """Assert kernel swap_in KV data matches naive_load_topk KV data."""
        naive_locs = self.coordinator.naive_load_topk(rpi, sls, batch, layer_id)
        for b in range(batch.shape[0]):
            for i in range(TOP_K):
                if batch[b, i] < 0:
                    continue
                naive_data = self.device_pool.kv_buffer[layer_id][
                    naive_locs[b, i].long()
                ]
                kernel_data = self.device_pool.kv_buffer[layer_id][
                    kernel_locs[b, i].long()
                ]
                self.assertTrue(
                    torch.allclose(naive_data.float(), kernel_data.float(), atol=1e-2),
                    f"{msg}layer {layer_id}, b{b} idx {i}: naive != kernel",
                )
```
**EN:** Assert kernel swap_in KV data matches naive_load_topk KV data. This block implements `_assert_matches_naive` and captures one focused piece of the module's behavior.
**CN:** Assert kernel swap_in KV data matches naive_load_topk KV data. 该代码块实现 `_assert_matches_naive`，承担模块行为中的一个聚焦逻辑片段。

### Lines 320-333: method swap in selected pages / 方法 swap in selected pages
```python
    def _swap_in_selected_pages(
        self,
        rpi: torch.Tensor,
        sls: torch.Tensor,
        batch: torch.Tensor,
        layer_id: int,
    ) -> torch.Tensor:
        """Wrapper that sets num_real_reqs before calling swap_in_selected_pages.

        In production, model_runner sets num_real_reqs before each forward
        pass.  Tests must replicate that to get correct kernel behaviour.
        """
        self.coordinator.num_real_reqs[0] = rpi.shape[0]
        return self.coordinator.swap_in_selected_pages(rpi, sls, batch, layer_id)
```
**EN:** Wrapper that sets num_real_reqs before calling swap_in_selected_pages. This block implements `_swap_in_selected_pages` and captures one focused piece of the module's behavior.
**CN:** Wrapper that sets num_real_reqs before calling swap_in_selected_pages. 该代码块实现 `_swap_in_selected_pages`，承担模块行为中的一个聚焦逻辑片段。

### Lines 335-342: method cleanup req / 方法 cleanup req
```python
    def _cleanup_req(self, req, kv_loc, *, logical_only=False):
        """request_finished -> free KV -> free req slot."""
        self.coordinator.request_finished(req)
        if logical_only:
            self.allocator.logical_attn_allocator.free(kv_loc)
        else:
            self.allocator.free(kv_loc)
        self._free_req_slot(req)
```
**EN:** request_finished -> free KV -> free req slot. This block implements `_cleanup_req` and captures one focused piece of the module's behavior.
**CN:** request_finished -> free KV -> free req slot. 该代码块实现 `_cleanup_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 344-350: method get initial sizes / 方法 get initial sizes
```python
    def _get_initial_sizes(self):
        """Snapshot allocator available sizes."""
        return (
            self.allocator.logical_attn_allocator.available_size(),
            self.allocator.hisparse_attn_allocator.available_size(),
            self.coordinator.mem_pool_host.available_size(),
        )
```
**EN:** Snapshot allocator available sizes. This block implements `_get_initial_sizes` and captures one focused piece of the module's behavior.
**CN:** Snapshot allocator available sizes. 该代码块实现 `_get_initial_sizes`，承担模块行为中的一个聚焦逻辑片段。

### Lines 352-357: method assert sizes restored / 方法 assert sizes restored
```python
    def _assert_sizes_restored(self, initial_sizes, msg=""):
        """Assert allocator sizes match the snapshot."""
        logical, hisparse, host = self._get_initial_sizes()
        self.assertEqual(logical, initial_sizes[0], f"Logical leak {msg}")
        self.assertEqual(hisparse, initial_sizes[1], f"HiSparse leak {msg}")
        self.assertEqual(host, initial_sizes[2], f"Host leak {msg}")
```
**EN:** Assert allocator sizes match the snapshot. This block implements `_assert_sizes_restored` and captures one focused piece of the module's behavior.
**CN:** Assert allocator sizes match the snapshot. 该代码块实现 `_assert_sizes_restored`，承担模块行为中的一个聚焦逻辑片段。

### Lines 358-361: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Test: Kernel correctness — short sequence (fast path)
    # ==================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 362-388: test case kernel correctness short seq / 测试用例 kernel correctness short seq
```python
    def test_kernel_correctness_short_seq(self):
        """Short seq (len <= device_buffer_size): kernel fast path returns
        device buffer locs, matching naive_load_topk."""
        initial = self._get_initial_sizes()
        req = _make_req("short-seq", list(range(self.page_size)))
        self._alloc_req_slot(req)

        fill_len = self.page_size
        kv_loc = self._alloc_kv(req, fill_len)
        self._write_device_patterns(kv_loc, fill_len)
        self.coordinator.alloc_device_buffer(req)

        tokens = self._build_topk_tokens(fill_len)
        batch = tokens.unsqueeze(0)
        rpi, sls = self._make_batch_tensors([req], [fill_len])

        for lid in range(LAYER_NUM):
            naive_locs = self.coordinator.naive_load_topk(rpi, sls, batch, lid)
            kernel_locs = self._swap_in_selected_pages(rpi, sls, batch, lid)
            valid = batch[0] >= 0
            self.assertTrue(
                torch.equal(naive_locs[0][valid].cpu(), kernel_locs[0][valid].cpu()),
                f"Layer {lid}: kernel locs != naive oracle",
            )

        self._cleanup_req(req, kv_loc)
        self._assert_sizes_restored(initial, "short_seq")
```
**EN:** Short seq (len <= device_buffer_size): kernel fast path returns device buffer locs, matching naive_load_topk. This test exercises `test_kernel_correctness_short_seq` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Short seq (len <= device_buffer_size): kernel fast path returns device buffer locs, matching naive_load_topk. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kernel_correctness_short_seq`。

### Lines 389-392: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Test: Kernel correctness — long sequence (cache miss + host DMA)
    # ==================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 393-422: test case kernel correctness long seq / 测试用例 kernel correctness long seq
```python
    def test_kernel_correctness_long_seq(self):
        """Long seq (len > device_buffer_size): kernel loads from host,
        matching naive_load_topk for data correctness."""
        initial = self._get_initial_sizes()
        fill_len = DEVICE_BUFFER_SIZE + self.page_size * 2
        req = _make_req("long-seq", list(range(fill_len)))
        self._alloc_req_slot(req)

        kv_loc = self._alloc_kv(req, fill_len, logical_only=True)
        self._populate_host_pool(req, fill_len)
        self.coordinator.admit_request_direct(req)

        # Pass fill_len-1 so position fill_len-1 ("newest token") is never
        # randomly selected — its reserved device-buffer slot is only valid
        # after map_last_loc_to_buffer in a real decode step.
        tokens = self._build_topk_tokens(fill_len - 1)
        batch = tokens.unsqueeze(0)
        rpi, sls = self._make_batch_tensors([req], [fill_len])

        for lid in range(LAYER_NUM):
            naive_locs = self.coordinator.naive_load_topk(rpi, sls, batch, lid)
            kernel_locs = self._swap_in_selected_pages(rpi, sls, batch, lid)
            self.assertTrue(torch.all(naive_locs[0, :TOP_K] >= 0))
            self.assertTrue(torch.all(kernel_locs[0, :TOP_K] >= 0))
            # Verify both return correct KV data independently
            self._assert_kv_correct(naive_locs[0], tokens, lid, TOP_K, msg="Naive: ")
            self._assert_kv_correct(kernel_locs[0], tokens, lid, TOP_K, msg="Kernel: ")

        self._cleanup_req(req, kv_loc, logical_only=True)
        self._assert_sizes_restored(initial, "long_seq")
```
**EN:** Long seq (len > device_buffer_size): kernel loads from host, matching naive_load_topk for data correctness. This test exercises `test_kernel_correctness_long_seq` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Long seq (len > device_buffer_size): kernel loads from host, matching naive_load_topk for data correctness. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kernel_correctness_long_seq`。

### Lines 423-426: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Test: Kernel LRU replacement across multiple decode steps
    # ==================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 427-481: test case kernel lru replacement / 测试用例 kernel lru replacement
```python
    def test_kernel_lru_replacement(self):
        """Multi-step swap-in: second call hits cached tokens, only
        evicts/loads new misses."""
        initial = self._get_initial_sizes()
        fill_len = DEVICE_BUFFER_SIZE + self.page_size * 2
        req = _make_req("lru-test", list(range(fill_len)))
        self._alloc_req_slot(req)

        kv_loc = self._alloc_kv(req, fill_len, logical_only=True)
        self._populate_host_pool(req, fill_len)
        self.coordinator.admit_request_direct(req)

        rpi, sls = self._make_batch_tensors([req], [fill_len])

        # Step 1: load the first TOP_K positions from host (no newest token —
        # the reserved slot is only valid after map_last_loc_to_buffer which is
        # called during an actual decode step, not modelled here).
        tokens_s1 = torch.arange(TOP_K, dtype=torch.int32, device="cuda")
        locs1 = self._swap_in_selected_pages(
            rpi, sls, tokens_s1.unsqueeze(0), layer_id=0
        )
        self.assertTrue(torch.all(locs1[0, :TOP_K] >= 0))

        # Step 2: half overlap (hit) + half new (miss).
        # Choose new tokens from a range safely below fill_len.
        half = TOP_K // 2
        new_start = TOP_K  # first position not in step-1
        tokens_s2 = torch.cat(
            [
                tokens_s1[:half],  # hits
                torch.arange(
                    new_start, new_start + half, dtype=torch.int32, device="cuda"
                ),  # misses
            ]
        )
        locs2 = self._swap_in_selected_pages(
            rpi, sls, tokens_s2.unsqueeze(0), layer_id=0
        )
        self.assertTrue(torch.all(locs2[0, :TOP_K] >= 0))

        # Verify repeated (hit) tokens still have correct KV data
        self._assert_kv_correct(
            locs2[0], tokens_s2, layer_id=0, count=half, msg="LRU hit: "
        )
        # Also verify new (miss) tokens loaded correctly
        self._assert_kv_correct(
            locs2[0, half:],
            tokens_s2[half:],
            layer_id=0,
            count=half,
            msg="LRU miss: ",
        )

        self._cleanup_req(req, kv_loc, logical_only=True)
        self._assert_sizes_restored(initial, "lru_replacement")
```
**EN:** Multi-step swap-in: second call hits cached tokens, only evicts/loads new misses. This test exercises `test_kernel_lru_replacement` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Multi-step swap-in: second call hits cached tokens, only evicts/loads new misses. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kernel_lru_replacement`。

### Lines 482-485: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Test: Allocator alloc/free lifecycle
    # ==================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 486-518: test case allocator alloc free cycle / 测试用例 allocator alloc free cycle
```python
    def test_allocator_alloc_free_cycle(self):
        """alloc_extend / alloc_device_buffer / free restores available_size."""
        initial = self._get_initial_sizes()
        device = self.allocator.device
        fill_len = self.page_size * 2

        kv_loc = self.allocator.alloc_extend(
            prefix_lens=torch.tensor([0], dtype=torch.int64, device=device),
            prefix_lens_cpu=torch.tensor([0], dtype=torch.int64),
            seq_lens=torch.tensor([fill_len], dtype=torch.int64, device=device),
            seq_lens_cpu=torch.tensor([fill_len], dtype=torch.int64),
            last_loc=torch.tensor([-1], dtype=torch.int64, device=device),
            extend_num_tokens=fill_len,
        )
        self.assertIsNotNone(kv_loc)
        self.assertEqual(len(kv_loc), fill_len)

        mapping = self.allocator.full_to_hisparse_device_index_mapping[kv_loc]
        self.assertTrue(torch.all(mapping > 0), "Mapping should be non-zero")
        self.assertLess(self.allocator.available_size(), initial[0])

        need_size = min(
            ((fill_len + self.page_size - 1) // self.page_size) * self.page_size,
            DEVICE_BUFFER_SIZE,
        )
        buf_idx = self.allocator.alloc_device_buffer(kv_loc, need_size)
        self.assertIsNotNone(buf_idx)
        mapping_after = self.allocator.full_to_hisparse_device_index_mapping[kv_loc]
        self.assertTrue(torch.all(mapping_after == 0), "Mapping should be cleared")

        self.allocator.free_hisparse_indices(buf_idx)
        self.allocator.logical_attn_allocator.free(kv_loc)
        self._assert_sizes_restored(initial, "alloc_free_cycle")
```
**EN:** alloc_extend / alloc_device_buffer / free restores available_size. This test exercises `test_allocator_alloc_free_cycle` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** alloc_extend / alloc_device_buffer / free restores available_size. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_allocator_alloc_free_cycle`。

### Lines 519-522: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Test: Staging (PD Colocate) path
    # ==================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 523-555: test case request lifecycle staging path / 测试用例 request lifecycle staging path
```python
    def test_request_lifecycle_staging_path(self):
        """prefill -> staging DMA -> collect_ready -> swap-in -> finish."""
        initial = self._get_initial_sizes()
        fill_len = self.page_size
        req = _make_req("staging-req", list(range(fill_len)))
        self._alloc_req_slot(req)

        kv_loc = self._alloc_kv(req, fill_len)
        self._write_device_patterns(kv_loc, fill_len)

        self.coordinator.admit_request_into_staging(req)
        self.assertTrue(req.hisparse_staging)

        torch.cuda.synchronize()
        ready = self.coordinator.collect_ready_reqs()
        self.assertEqual(len(ready), 1)
        self.assertFalse(req.hisparse_staging)
        self.assertTrue(self.coordinator._skip_first_backup[req.req_pool_idx])

        tokens = self._build_topk_tokens(fill_len)
        batch = tokens.unsqueeze(0)
        rpi, sls = self._make_batch_tensors([req], [fill_len])

        locs = self._swap_in_selected_pages(rpi, sls, batch, layer_id=0)
        valid_n = min(fill_len, TOP_K)
        self.assertTrue(torch.all(locs[0, :valid_n] >= 0))
        self._assert_kv_correct(
            locs[0], tokens, layer_id=0, count=valid_n, msg="Staging: "
        )
        self._assert_matches_naive(rpi, sls, batch, locs, layer_id=0, msg="Staging: ")

        self._cleanup_req(req, kv_loc)
        self._assert_sizes_restored(initial, "staging_path")
```
**EN:** prefill -> staging DMA -> collect_ready -> swap-in -> finish. This test exercises `test_request_lifecycle_staging_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** prefill -> staging DMA -> collect_ready -> swap-in -> finish. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_request_lifecycle_staging_path`。

### Lines 556-559: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Test: Direct-to-host (PD separated) path
    # ==================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 560-590: test case request lifecycle direct path / 测试用例 request lifecycle direct path
```python
    def test_request_lifecycle_direct_path(self):
        """alloc_logical_only -> host write -> admit_direct -> swap-in -> finish."""
        initial = self._get_initial_sizes()
        fill_len = DEVICE_BUFFER_SIZE + self.page_size
        req = _make_req("direct-req", list(range(fill_len)))
        self._alloc_req_slot(req)

        kv_loc = self._alloc_kv(req, fill_len, logical_only=True)
        self._populate_host_pool(req, fill_len)
        self.coordinator.admit_request_direct(req)

        self.assertFalse(req.staging)
        self.assertTrue(self.coordinator._skip_first_backup[req.req_pool_idx])
        buf_tokens = self.coordinator.req_device_buffer_tokens[
            :, req.req_pool_idx, :DEVICE_BUFFER_SIZE
        ]
        self.assertTrue(torch.all(buf_tokens == -1))

        tokens = self._build_topk_tokens(fill_len - 1)
        batch = tokens.unsqueeze(0)
        rpi, sls = self._make_batch_tensors([req], [fill_len])

        locs = self._swap_in_selected_pages(rpi, sls, batch, layer_id=0)
        self.assertTrue(torch.all(locs[0, :TOP_K] >= 0))
        self._assert_kv_correct(
            locs[0], tokens, layer_id=0, count=TOP_K, msg="Direct: "
        )
        self._assert_matches_naive(rpi, sls, batch, locs, layer_id=0, msg="Direct: ")

        self._cleanup_req(req, kv_loc, logical_only=True)
        self._assert_sizes_restored(initial, "direct_path")
```
**EN:** alloc_logical_only -> host write -> admit_direct -> swap-in -> finish. This test exercises `test_request_lifecycle_direct_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** alloc_logical_only -> host write -> admit_direct -> swap-in -> finish. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_request_lifecycle_direct_path`。

### Lines 591-594: supporting source context / 辅助源码上下文
```python

    # ==================================================================
    # Test: Batch multiple requests
    # ==================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 595-647: test case batch multiple requests / 测试用例 batch multiple requests
```python
    def test_batch_multiple_requests(self):
        """Mix of short & long requests in batch: kernel correct + no leaks."""
        initial = self._get_initial_sizes()

        configs = [
            ("batch-short-0", self.page_size),
            ("batch-short-1", self.page_size),
            ("batch-long-0", DEVICE_BUFFER_SIZE + self.page_size),
            ("batch-long-1", DEVICE_BUFFER_SIZE + self.page_size * 2),
        ]

        reqs, kv_locs = [], []
        for rid, fl in configs:
            req = _make_req(rid, list(range(fl)))
            self._alloc_req_slot(req)
            is_long = fl > DEVICE_BUFFER_SIZE
            kv_loc = self._alloc_kv(req, fl, logical_only=is_long)
            if is_long:
                self._populate_host_pool(req, fl)
                self.coordinator.admit_request_direct(req)
            else:
                self._write_device_patterns(kv_loc, fl)
                self.coordinator.alloc_device_buffer(req)
            reqs.append(req)
            kv_locs.append(kv_loc)

        rpi, sls = self._make_batch_tensors(reqs, [c[1] for c in configs])
        top_k_batch = torch.stack(
            [
                # For long sequences pass fl-1 to exclude the "newest token" position
                # whose reserved device-buffer slot is not populated in unit tests.
                self._build_topk_tokens(fl - 1 if fl > DEVICE_BUFFER_SIZE else fl)
                for _, fl in configs
            ]
        )

        for lid in range(LAYER_NUM):
            locs = self._swap_in_selected_pages(rpi, sls, top_k_batch, lid)
            for i, (rid, fl) in enumerate(configs):
                vn = min(fl, TOP_K)
                self.assertTrue(
                    torch.all(locs[i, :vn] >= 0),
                    f"Req {rid}, layer {lid}: negative locs",
                )
                self._assert_kv_correct(
                    locs[i], top_k_batch[i], lid, vn, msg=f"{rid}: "
                )

        for i, req in enumerate(reqs):
            is_long = configs[i][1] > DEVICE_BUFFER_SIZE
            self._cleanup_req(req, kv_locs[i], logical_only=is_long)

        self._assert_sizes_restored(initial, "batch_multiple")
```
**EN:** Mix of short & long requests in batch: kernel correct + no leaks. This test exercises `test_batch_multiple_requests` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Mix of short & long requests in batch: kernel correct + no leaks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_multiple_requests`。

### Lines 650-651: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_req`: Create a minimal mock Req object with the fields HiSparseCoordinator uses. / 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiSparseUnit`: Test class that builds a minimal HiSparse component stack. / 用于组织相关测试、夹具或辅助方法。
- `TestHiSparseUnit.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestHiSparseUnit.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestHiSparseUnit.setUp`: Reset shared allocator / coordinator state so tests are isolated. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestHiSparseUnit._alloc_req_slot`: Allocate a req_pool_idx for the request. / 该代码块实现 `_alloc_req_slot`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiSparseUnit._free_req_slot`: Free the req_pool_idx. / 该代码块实现 `_free_req_slot`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiSparseUnit._alloc_kv`: Allocate KV indices, write req_to_token_pool, update req fields. / 该代码块实现 `_alloc_kv`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiSparseUnit._kv_pattern`: Deterministic KV value for (layer, token) — used by write & verify. / 该代码块实现 `_kv_pattern`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiSparseUnit._write_device_patterns`: Write distinguishable patterns into device KV buffer for all layers. / 该代码块实现 `_write_device_patterns`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiSparseUnit._populate_host_pool`: Allocate host slots, write known patterns, register in coordinator. / 该代码块实现 `_populate_host_pool`，承担模块行为中的一个聚焦逻辑片段。
- `TestHiSparseUnit._build_topk_tokens`: Build a 1-D [TOP_K] int32 cuda tensor of token positions. / 该代码块实现 `_build_topk_tokens`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `unittest`, `types`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 651
