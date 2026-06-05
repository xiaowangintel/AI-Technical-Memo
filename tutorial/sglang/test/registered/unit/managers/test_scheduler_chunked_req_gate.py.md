# test_scheduler_chunked_req_gate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_scheduler_chunked_req_gate.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates scheduler chunked req gate behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 scheduler chunked req gate 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Regression tests for the SWA chunked-req stash gate (#24252)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-10: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from unittest.mock import MagicMock

import torch

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase, maybe_stub_sgl_kernel
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `unittest.mock`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `unittest.mock`, `torch`。

### Lines 12-12: supporting statements / 辅助语句
```python
maybe_stub_sgl_kernel()
```
**EN:** This block performs supporting work through calls such as `maybe_stub_sgl_kernel`, preparing state for nearby definitions.
**CN:** 该代码块通过 `maybe_stub_sgl_kernel` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 14-16: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.managers.schedule_batch import Req
from sglang.srt.managers.scheduler import Scheduler
from sglang.srt.mem_cache.chunk_cache import ChunkCache
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.scheduler`, `sglang.srt.mem_cache.chunk_cache`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.scheduler`, `sglang.srt.mem_cache.chunk_cache`。

### Lines 18-18: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-49: function make req / 函数 make req
```python
def _make_req(
    *,
    req_pool_idx: int,
    fill_ids: list,
    prefix_indices: torch.Tensor,
    extend_input_len: int,
) -> Req:
    req = Req.__new__(Req)
    req.rid = "test-req"
    req.origin_input_ids = list(fill_ids)
    req.output_ids = []
    req.fill_ids = list(fill_ids)
    req.prefix_indices = prefix_indices
    req.req_pool_idx = req_pool_idx
    req.extend_input_len = extend_input_len
    req.is_chunked = 0
    req.host_hit_length = 0
    req.cache_protected_len = 0
    req.skip_radix_cache_insert = False
    req.last_node = None
    req.swa_uuid_for_lock = None
    req.session = None
    req.return_logprob = False
    req.logprob_start_len = -1
    req.positional_embed_overrides = None
    req.extra_key = None
    req.mamba_pool_idx = None
    req.sampling_params = SimpleNamespace(max_new_tokens=128, ignore_eos=False)
    return req
```
**EN:** This block implements `_make_req` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 52-60: function make req to token pool / 函数 make req to token pool
```python
def _make_req_to_token_pool(num_slots: int, max_context: int) -> SimpleNamespace:
    # Slot s contains a recognizable fingerprint [s*1000, s*1000+1, ...]
    # so we can tell a corrupted prefix_indices from a healthy one by content.
    pool = SimpleNamespace()
    pool.req_to_token = (
        torch.arange(max_context, dtype=torch.int32).unsqueeze(0).repeat(num_slots, 1)
        + torch.arange(num_slots, dtype=torch.int32).unsqueeze(1) * 1000
    )
    return pool
```
**EN:** This block implements `_make_req_to_token_pool` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_req_to_token_pool`，承担模块行为中的一个聚焦逻辑片段。

### Lines 63-70: function make chunk cache / 函数 make chunk cache
```python
def _make_chunk_cache(req_to_token_pool) -> ChunkCache:
    return ChunkCache(
        SimpleNamespace(
            req_to_token_pool=req_to_token_pool,
            token_to_kv_pool_allocator=None,
            page_size=1,
        )
    )
```
**EN:** This block implements `_make_chunk_cache` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_chunk_cache`，承担模块行为中的一个聚焦逻辑片段。

### Lines 73-99: function scheduler for get next batch / 函数 scheduler for get next batch
```python
def _scheduler_for_get_next_batch(*, tree_cache, chunked_req) -> Scheduler:
    s = Scheduler.__new__(Scheduler)
    s._abort_on_waiting_timeout = MagicMock()
    s._abort_on_running_timeout = MagicMock()
    s.dllm_config = None
    s.dllm_manager = None
    s.enable_hisparse = False
    s.enable_fpm = False
    s.last_batch = None
    s.require_mlp_sync = False
    s.spec_algorithm = MagicMock()
    s.server_args = MagicMock(speculative_skip_dp_mlp_sync=True)
    s.running_batch = MagicMock()
    s.running_batch.is_empty.return_value = True
    s.running_batch.is_prefill_only = False
    s.running_batch.batch_is_full = False
    s.running_batch.reqs = []
    s.get_new_batch_prefill = MagicMock(return_value=None)
    s.dp_attn_adapter = MagicMock()
    s.dp_attn_adapter.maybe_prepare_mlp_sync_batch = MagicMock(
        side_effect=lambda batch, **_: batch
    )
    s._maybe_prepare_ngram_embedding = MagicMock(side_effect=lambda batch: batch)
    s.update_running_batch = MagicMock(side_effect=lambda batch: batch)
    s.tree_cache = tree_cache
    s.chunked_req = chunked_req
    return s
```
**EN:** This block implements `_scheduler_for_get_next_batch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_scheduler_for_get_next_batch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 102-102: class TestStashGatePreservesPrefixIndices declaration / 类 TestStashGatePreservesPrefixIndices 声明
```python
class TestStashGatePreservesPrefixIndices(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 103-111: class-level constants and configuration for `TestStashGatePreservesPrefixIndices` / 类级常量与配置
```python
    """Consumer side: real ChunkCache.cache_unfinished_req mutates
    req.prefix_indices iff stash actually runs, so prefix_indices content
    is the bug-detection signal."""

    POOL_IDX = 4
    INITIAL_PREFIX_LEN = 8  # what was really cached last iter
    POST_RESET_FILL_LEN = 32  # length after init_next_round_input
    NUM_SLOTS = 8
    MAX_CONTEXT = 64
```
**EN:** This block defines shared names such as `POOL_IDX`, `INITIAL_PREFIX_LEN`, `POST_RESET_FILL_LEN`, `NUM_SLOTS`, `MAX_CONTEXT`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `POOL_IDX`, `INITIAL_PREFIX_LEN`, `POST_RESET_FILL_LEN`, `NUM_SLOTS`, `MAX_CONTEXT` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 113-127: method build / 方法 build
```python
    def _build(self, flag: bool):
        pool = _make_req_to_token_pool(self.NUM_SLOTS, self.MAX_CONTEXT)
        cache = _make_chunk_cache(pool)
        initial_prefix = pool.req_to_token[self.POOL_IDX, : self.INITIAL_PREFIX_LEN].to(
            dtype=torch.int64, copy=True
        )
        req = _make_req(
            req_pool_idx=self.POOL_IDX,
            fill_ids=list(range(self.POST_RESET_FILL_LEN)),
            prefix_indices=initial_prefix,
            extend_input_len=0,
        )
        s = _scheduler_for_get_next_batch(tree_cache=cache, chunked_req=req)
        s._chunked_req_scheduled_last_iter = flag
        return s, req, initial_prefix, pool
```
**EN:** This block implements `_build` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build`，承担模块行为中的一个聚焦逻辑片段。

### Lines 129-137: test case deferred chunked req keeps real prefix indices / 测试用例 deferred chunked req keeps real prefix indices
```python
    def test_deferred_chunked_req_keeps_real_prefix_indices(self):
        # The bug case: a spurious stash on a deferred chunked_req
        # would extend prefix_indices to len(fill_ids).
        s, req, initial_prefix, _ = self._build(flag=False)

        Scheduler.get_next_batch_to_run(s)

        self.assertEqual(req.prefix_indices.shape[0], self.INITIAL_PREFIX_LEN)
        self.assertTrue(torch.equal(req.prefix_indices, initial_prefix))
```
**EN:** This test exercises `test_deferred_chunked_req_keeps_real_prefix_indices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deferred_chunked_req_keeps_real_prefix_indices`。

### Lines 139-150: test case scheduled chunked req advances prefix indices via real stash / 测试用例 scheduled chunked req advances prefix indices via real stash
```python
    def test_scheduled_chunked_req_advances_prefix_indices_via_real_stash(self):
        # Symmetric guard against over-gating: when the chunked_req was
        # actually scheduled, stash must run and advance prefix_indices.
        s, req, _, pool = self._build(flag=True)

        Scheduler.get_next_batch_to_run(s)

        expected = pool.req_to_token[self.POOL_IDX, : self.POST_RESET_FILL_LEN].to(
            dtype=torch.int64
        )
        self.assertEqual(req.prefix_indices.shape[0], self.POST_RESET_FILL_LEN)
        self.assertTrue(torch.equal(req.prefix_indices, expected))
```
**EN:** This test exercises `test_scheduled_chunked_req_advances_prefix_indices_via_real_stash` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scheduled_chunked_req_advances_prefix_indices_via_real_stash`。

### Lines 152-161: test case no chunked req never mutates state even with stale flag / 测试用例 no chunked req never mutates state even with stale flag
```python
    def test_no_chunked_req_never_mutates_state_even_with_stale_flag(self):
        # Retract path clears chunked_req without resetting the flag;
        # the outer `if chunked_req is not None` guard must hold.
        pool = _make_req_to_token_pool(self.NUM_SLOTS, self.MAX_CONTEXT)
        cache = _make_chunk_cache(pool)
        s = _scheduler_for_get_next_batch(tree_cache=cache, chunked_req=None)
        s._chunked_req_scheduled_last_iter = True

        Scheduler.get_next_batch_to_run(s)
        self.assertIsNone(s.chunked_req)
```
**EN:** This test exercises `test_no_chunked_req_never_mutates_state_even_with_stale_flag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_chunked_req_never_mutates_state_even_with_stale_flag`。

### Lines 164-165: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_req`: This block implements `_make_req` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_req`，承担模块行为中的一个聚焦逻辑片段。
- `_make_req_to_token_pool`: This block implements `_make_req_to_token_pool` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_req_to_token_pool`，承担模块行为中的一个聚焦逻辑片段。
- `_make_chunk_cache`: This block implements `_make_chunk_cache` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_chunk_cache`，承担模块行为中的一个聚焦逻辑片段。
- `_scheduler_for_get_next_batch`: This block implements `_scheduler_for_get_next_batch` and captures one focused piece of the module's behavior. / 该代码块实现 `_scheduler_for_get_next_batch`，承担模块行为中的一个聚焦逻辑片段。
- `TestStashGatePreservesPrefixIndices`: Consumer side: real ChunkCache.cache_unfinished_req mutates req.prefix_indices iff stash actually runs, so prefix_indices content is the bug-detection signal. / 用于组织相关测试、夹具或辅助方法。
- `TestStashGatePreservesPrefixIndices._build`: This block implements `_build` and captures one focused piece of the module's behavior. / 该代码块实现 `_build`，承担模块行为中的一个聚焦逻辑片段。
- `TestStashGatePreservesPrefixIndices.test_deferred_chunked_req_keeps_real_prefix_indices`: This test exercises `test_deferred_chunked_req_keeps_real_prefix_indices` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deferred_chunked_req_keeps_real_prefix_indices`。
- `TestStashGatePreservesPrefixIndices.test_scheduled_chunked_req_advances_prefix_indices_via_real_stash`: This test exercises `test_scheduled_chunked_req_advances_prefix_indices_via_real_stash` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scheduled_chunked_req_advances_prefix_indices_via_real_stash`。
- `TestStashGatePreservesPrefixIndices.test_no_chunked_req_never_mutates_state_even_with_stale_flag`: This test exercises `test_no_chunked_req_never_mutates_state_even_with_stale_flag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_chunked_req_never_mutates_state_even_with_stale_flag`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.scheduler`, `sglang.srt.mem_cache.chunk_cache`

- **Total lines / 总行数**: 165
