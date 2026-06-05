# test_prefill_adder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_prefill_adder.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates prefill adder behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 prefill adder 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from unittest.mock import MagicMock

from sglang.srt.managers.schedule_batch import Req
from sglang.srt.managers.schedule_policy import AddReqResult, PrefillAdder
from sglang.srt.mem_cache.base_prefix_cache import (
    DecLockRefResult,
    IncLockRefResult,
)
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `unittest.mock`, `sglang.srt.managers.schedule_batch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `unittest.mock`, `sglang.srt.managers.schedule_batch`。

### Lines 15-16: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=9, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=2, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 19-19: class TestPrefillAdder declaration / 类 TestPrefillAdder 声明
```python
class TestPrefillAdder(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 20-23: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))
        self.mock_tree_cache = self.create_tree_cache()
        self.mock_token_allocator = self.create_token_allocator()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 25-39: helper routine create tree cache / 辅助流程 create tree cache
```python
    def create_tree_cache(
        self,
        *,
        full_evictable_size: int = 0,
        swa_evictable_size: int = 0,
        evictable_size: int = 0,
    ) -> MagicMock:
        tree_cache = MagicMock()
        tree_cache.full_evictable_size.return_value = full_evictable_size
        tree_cache.swa_evictable_size.return_value = swa_evictable_size
        tree_cache.evictable_size.return_value = evictable_size
        tree_cache.disable = False
        tree_cache.inc_lock_ref.return_value = IncLockRefResult()
        tree_cache.dec_lock_ref.return_value = DecLockRefResult()
        return tree_cache
```
**EN:** This helper encapsulates `create_tree_cache` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_tree_cache`，以便周围测试复用准备、执行或校验逻辑。

### Lines 41-52: helper routine create token allocator / 辅助流程 create token allocator
```python
    def create_token_allocator(
        self,
        *,
        full_available_size: int = 0,
        swa_available_size: int = 0,
        available_size: int = 0,
    ) -> MagicMock:
        allocator = MagicMock()
        allocator.full_available_size.return_value = full_available_size
        allocator.swa_available_size.return_value = swa_available_size
        allocator.available_size.return_value = available_size
        return allocator
```
**EN:** This helper encapsulates `create_token_allocator` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_token_allocator`，以便周围测试复用准备、执行或校验逻辑。

### Lines 54-59: helper routine create running batch / 辅助流程 create running batch
```python
    def create_running_batch(self, reqs=None) -> MagicMock:
        batch = MagicMock()
        batch.reqs = list(reqs or [])
        batch.release_req.return_value = None
        batch.filter_batch.return_value = None
        return batch
```
**EN:** This helper encapsulates `create_running_batch` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_running_batch`，以便周围测试复用准备、执行或校验逻辑。

### Lines 61-68: helper routine create server args / 辅助流程 create server args
```python
    def create_server_args(
        self, *, schedule_low_priority_values_first: bool
    ) -> MagicMock:
        server_args = MagicMock()
        server_args.schedule_low_priority_values_first = (
            schedule_low_priority_values_first
        )
        return server_args
```
**EN:** This helper encapsulates `create_server_args` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_server_args`，以便周围测试复用准备、执行或校验逻辑。

### Lines 70-80: helper routine create mock req / 辅助流程 create mock req
```python
    def create_mock_req(self, rid, priority, max_new_tokens, output_len=0, wait_time=0):
        req = MagicMock(spec=Req)
        req.rid = str(rid)
        req.priority = priority
        req.extend_input_len = 0
        req.extend_logprob_start_len = 0
        req.output_ids = [0] * output_len
        req.sampling_params = SimpleNamespace(max_new_tokens=max_new_tokens)
        req.time_stats = SimpleNamespace(wait_queue_entry_time=wait_time)
        req.finished.return_value = False
        return req
```
**EN:** This helper encapsulates `create_mock_req` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_mock_req`，以便周围测试复用准备、执行或校验逻辑。

### Lines 82-95: helper routine create adder / 辅助流程 create adder
```python
    def create_adder(self, running_batch, **kwargs):
        defaults = dict(
            page_size=1,
            tree_cache=self.mock_tree_cache,
            token_to_kv_pool_allocator=self.mock_token_allocator,
            running_batch=running_batch,
            new_token_ratio=1.0,
            rem_input_tokens=10000,
            rem_chunk_tokens=None,
            num_mixed_decode_tokens=0,
            priority_scheduling_preemption_threshold=0,
        )
        defaults.update(kwargs)
        return PrefillAdder(**defaults)
```
**EN:** This helper encapsulates `create_adder` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_adder`，以便周围测试复用准备、执行或校验逻辑。

### Lines 97-127: test case preempt success high priority values first / 测试用例 preempt success high priority values first
```python
    def test_preempt_success_high_priority_values_first(self):
        params = [
            ("run1", 0, 50),
            ("run2", 1, 75),
            ("run3", 2, 100),
        ]
        running_reqs = [
            self.create_mock_req(rid, priority, max_new_tokens)
            for rid, priority, max_new_tokens in params
        ]
        mock_server_args = self.create_server_args(
            schedule_low_priority_values_first=False
        )
        running_batch = self.create_running_batch(running_reqs)
        adder = self.create_adder(running_batch)

        self.assertEqual(adder.rem_total_token_offset, 225)

        self.mock_token_allocator.full_available_size.return_value = (
            225  # full occupation of GRam
        )
        self.mock_token_allocator.available_size.return_value = 225

        new_req = self.create_mock_req("new1", priority=1, max_new_tokens=49)

        success = adder.preempt_to_schedule(new_req, mock_server_args)

        self.assertTrue(success)
        self.assertIn(running_reqs[0], adder.preempt_list)
        self.assertEqual(adder.rem_total_token_offset, 175)  # 50 + 75 + 100 - 50 = 175
        running_batch.release_req.assert_called_once()
```
**EN:** This test exercises `test_preempt_success_high_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_success_high_priority_values_first`。

### Lines 129-159: test case preempt success low priority values first / 测试用例 preempt success low priority values first
```python
    def test_preempt_success_low_priority_values_first(self):
        params = [
            ("run1", 0, 50),
            ("run2", 1, 75),
            ("run3", 2, 100),
        ]
        running_reqs = [
            self.create_mock_req(rid, priority, max_new_tokens)
            for rid, priority, max_new_tokens in params
        ]
        mock_server_args = self.create_server_args(
            schedule_low_priority_values_first=True
        )
        running_batch = self.create_running_batch(running_reqs)
        adder = self.create_adder(running_batch)

        self.assertEqual(adder.rem_total_token_offset, 225)

        self.mock_token_allocator.full_available_size.return_value = (
            225  # full occupation of GRam
        )
        self.mock_token_allocator.available_size.return_value = 225

        new_req = self.create_mock_req("new1", priority=1, max_new_tokens=49)

        success = adder.preempt_to_schedule(new_req, mock_server_args)

        self.assertTrue(success)
        self.assertIn(running_reqs[2], adder.preempt_list)
        self.assertEqual(adder.rem_total_token_offset, 125)  # 50 + 75 + 100 - 100 = 125
        running_batch.release_req.assert_called_once()
```
**EN:** This test exercises `test_preempt_success_low_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_success_low_priority_values_first`。

### Lines 161-199: test case preempt fail low priority values first / 测试用例 preempt fail low priority values first
```python
    def test_preempt_fail_low_priority_values_first(self):
        params = [
            ("run1", 0, 50),
            ("run2", 1, 75),
            ("run3", 2, 100),
        ]
        running_reqs = [
            self.create_mock_req(rid, priority, max_new_tokens)
            for rid, priority, max_new_tokens in params
        ]
        mock_server_args = self.create_server_args(
            schedule_low_priority_values_first=True
        )
        running_batch = self.create_running_batch(running_reqs)
        adder = self.create_adder(running_batch)

        self.assertEqual(adder.rem_total_token_offset, 225)

        self.mock_token_allocator.full_available_size.return_value = (
            225  # full occupation of GRam
        )
        self.mock_token_allocator.available_size.return_value = 225

        new_req_fail_by_priority_check = self.create_mock_req(
            "new1", priority=2, max_new_tokens=49
        )

        success_by_priority_check = adder.preempt_to_schedule(
            new_req_fail_by_priority_check, mock_server_args
        )
        self.assertFalse(success_by_priority_check)

        new_req_fail_by_priority_check = self.create_mock_req(
            "new2", priority=1, max_new_tokens=110
        )
        success_by_capacity_check = adder.preempt_to_schedule(
            new_req_fail_by_priority_check, mock_server_args
        )
        self.assertFalse(success_by_capacity_check)
```
**EN:** This test exercises `test_preempt_fail_low_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_fail_low_priority_values_first`。

### Lines 201-239: test case preempt fail high priority values first / 测试用例 preempt fail high priority values first
```python
    def test_preempt_fail_high_priority_values_first(self):
        params = [
            ("run1", 0, 50),
            ("run2", 1, 75),
            ("run3", 2, 100),
        ]
        running_reqs = [
            self.create_mock_req(rid, priority, max_new_tokens)
            for rid, priority, max_new_tokens in params
        ]
        mock_server_args = self.create_server_args(
            schedule_low_priority_values_first=False
        )
        running_batch = self.create_running_batch(running_reqs)
        adder = self.create_adder(running_batch)

        self.assertEqual(adder.rem_total_token_offset, 225)

        self.mock_token_allocator.full_available_size.return_value = (
            225  # full occupation of GRam
        )
        self.mock_token_allocator.available_size.return_value = 225

        new_req_fail_by_priority_check = self.create_mock_req(
            "new1", priority=0, max_new_tokens=49
        )

        success_by_priority_check = adder.preempt_to_schedule(
            new_req_fail_by_priority_check, mock_server_args
        )
        self.assertFalse(success_by_priority_check)

        new_req_fail_by_priority_check = self.create_mock_req(
            "new2", priority=-1, max_new_tokens=110
        )
        success_by_capacity_check = adder.preempt_to_schedule(
            new_req_fail_by_priority_check, mock_server_args
        )
        self.assertFalse(success_by_capacity_check)
```
**EN:** This test exercises `test_preempt_fail_high_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_fail_high_priority_values_first`。

### Lines 241-282: test case preempt skip already preempted request / 测试用例 preempt skip already preempted request
```python
    def test_preempt_skip_already_preempted_request(self):
        params = [
            ("req_prio_0", 0, 50),
            ("req_prio_1", 1, 75),
            ("req_prio_2", 2, 100),
        ]
        running_reqs = [
            self.create_mock_req(rid, priority, max_new_tokens)
            for rid, priority, max_new_tokens in params
        ]
        mock_server_args = self.create_server_args(
            schedule_low_priority_values_first=False
        )
        running_batch = self.create_running_batch(running_reqs)
        adder = self.create_adder(running_batch)

        self.assertEqual(adder.rem_total_token_offset, 225)

        self.mock_token_allocator.full_available_size.return_value = 225
        self.mock_token_allocator.available_size.return_value = 225

        # New request preempts req_prio_0
        first_req = self.create_mock_req(
            "new_req_prio_1", priority=1, max_new_tokens=49
        )
        first_success = adder.preempt_to_schedule(first_req, mock_server_args)
        self.assertTrue(first_success)
        self.assertIn(running_reqs[0], adder.preempt_list)
        self.assertEqual(adder.rem_total_token_offset, 175)
        running_batch.release_req.assert_called_once()

        # Second call needs more tokens than currently free, so it would need to
        # preempt req_prio_0 again if already-preempted requests were not filtered out.
        second_req = self.create_mock_req(
            "second_new_req_prio_1", priority=1, max_new_tokens=76
        )
        second_success = adder.preempt_to_schedule(second_req, mock_server_args)

        self.assertFalse(second_success)
        self.assertEqual(adder.rem_total_token_offset, 175)
        self.assertEqual(adder.preempt_list.count(running_reqs[0]), 1)
        running_batch.release_req.assert_called_once()
```
**EN:** This test exercises `test_preempt_skip_already_preempted_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_skip_already_preempted_request`。

### Lines 284-317: test case preempt success low priority values first exact once / 测试用例 preempt success low priority values first exact once
```python
    def test_preempt_success_low_priority_values_first_exact_once(self):
        params = [
            ("run1", 0, 50),
            ("run2", 1, 75),
            ("run3", 2, 100),
            ("run4", 2, 125),
            ("run4", 2, 125),
        ]
        running_reqs = [
            self.create_mock_req(rid, priority, max_new_tokens)
            for rid, priority, max_new_tokens in params
        ]
        mock_server_args = self.create_server_args(
            schedule_low_priority_values_first=True
        )
        running_batch = self.create_running_batch(running_reqs)
        adder = self.create_adder(running_batch)

        self.assertEqual(adder.rem_total_token_offset, 475)

        self.mock_token_allocator.full_available_size.return_value = (
            475  # full occupation of GRam
        )
        self.mock_token_allocator.available_size.return_value = 475

        new_req = self.create_mock_req("new1", priority=1, max_new_tokens=75)

        success = adder.preempt_to_schedule(new_req, mock_server_args)
        self.assertTrue(success)
        self.assertIn(running_reqs[2], adder.preempt_list)
        self.assertEqual(
            adder.rem_total_token_offset, 375
        )  # 50 + 75 + 100 + 125 + 125 - 100 = 375
        running_batch.release_req.assert_called_once()
```
**EN:** This test exercises `test_preempt_success_low_priority_values_first_exact_once` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_success_low_priority_values_first_exact_once`。

### Lines 319-353: test case preempt success low priority values first exact twice / 测试用例 preempt success low priority values first exact twice
```python
    def test_preempt_success_low_priority_values_first_exact_twice(self):
        params = [
            ("run1", 0, 50),
            ("run2", 1, 75),
            ("run3", 2, 100),
            ("run4", 2, 125),
            ("run4", 2, 125),
        ]
        running_reqs = [
            self.create_mock_req(rid, priority, max_new_tokens)
            for rid, priority, max_new_tokens in params
        ]
        mock_server_args = self.create_server_args(
            schedule_low_priority_values_first=True
        )
        running_batch = self.create_running_batch(running_reqs)
        adder = self.create_adder(running_batch)

        self.assertEqual(adder.rem_total_token_offset, 475)

        self.mock_token_allocator.full_available_size.return_value = (
            475  # full occupation of GRam
        )
        self.mock_token_allocator.available_size.return_value = 475

        new_req = self.create_mock_req("new1", priority=1, max_new_tokens=200)

        success = adder.preempt_to_schedule(new_req, mock_server_args)
        self.assertTrue(success)
        self.assertIn(running_reqs[2], adder.preempt_list)
        self.assertIn(running_reqs[3], adder.preempt_list)
        self.assertEqual(
            adder.rem_total_token_offset, 250
        )  # 50 + 75 + 100 + 125 + 125 - 100 - 125 = 250
        self.assertEqual(running_batch.release_req.call_count, 2)
```
**EN:** This test exercises `test_preempt_success_low_priority_values_first_exact_twice` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_success_low_priority_values_first_exact_twice`。

### Lines 355-434: test case mixed chunk prefill budgets (part 1/2) / 测试用例 mixed chunk prefill budgets（第 1/2 部分）
```python
    def test_mixed_chunk_prefill_budgets(self):
        self.mock_token_allocator.available_size.return_value = 1000

        decode_reqs = [
            self.create_mock_req(f"decode_{i}", priority=0, max_new_tokens=50)
            for i in range(8)
        ]
        running_batch = self.create_running_batch(decode_reqs)

        adder = self.create_adder(
            running_batch,
            rem_input_tokens=200,
            rem_chunk_tokens=64,
            num_mixed_decode_tokens=len(decode_reqs),
        )

        self.assertEqual(adder.rem_input_tokens, 192)  # 200 - 8
        self.assertEqual(adder.rem_chunk_tokens, 56)  # 64 - 8
        self.assertEqual(adder.rem_total_token_offset, 408)  # 8 + 8 * 50
        self.assertEqual(adder.cur_rem_token_offset, 8)
        self.assertEqual(adder.budget_state(), AddReqResult.CONTINUE)

        # Add a prefill that exactly consumes the chunk budget
        req1 = self.create_mock_req("req1", priority=0, max_new_tokens=64)
        req1.extend_input_len = 56
        req1.host_hit_length = 0
        req1.prefix_indices = []
        req1.fill_ids = list(range(56))
        req1.last_node = MagicMock()
        req1.sampling_params.ignore_eos = False

        result1 = adder.add_one_req(
            req1, has_chunked_req=False, truncation_align_size=None
        )

        self.assertEqual(len(adder.can_run_list), 1)
        self.assertEqual(adder.rem_chunk_tokens, 0)  # 56 - 56
        self.assertEqual(adder.rem_input_tokens, 136)  # 192 - 56
        self.assertEqual(result1, AddReqResult.OTHER)

        # 3 decode requests finished
        remaining_decode_reqs = decode_reqs[3:]
        running_batch2 = self.create_running_batch(remaining_decode_reqs)

        adder2 = self.create_adder(
            running_batch2,
            rem_input_tokens=200,
            rem_chunk_tokens=64,
            num_mixed_decode_tokens=len(remaining_decode_reqs),
        )

        self.assertEqual(adder2.rem_input_tokens, 195)  # 200 - 5
        self.assertEqual(adder2.rem_chunk_tokens, 59)  # 64 - 5
        self.assertEqual(adder2.rem_total_token_offset, 255)  # 5 + 5 * 50
        self.assertEqual(adder2.budget_state(), AddReqResult.CONTINUE)

        # Same prefill no longer exhausts the chunk budget
        req2 = self.create_mock_req("req2", priority=0, max_new_tokens=64)
        req2.extend_input_len = 56
        req2.host_hit_length = 0
        req2.prefix_indices = []
        req2.fill_ids = list(range(56))
        req2.last_node = MagicMock()
        req2.sampling_params.ignore_eos = False

        result2 = adder2.add_one_req(
            req2, has_chunked_req=False, truncation_align_size=None
        )

        self.assertEqual(len(adder2.can_run_list), 1)
        self.assertEqual(adder2.rem_chunk_tokens, 3)  # 59 - 56 = 3 remaining
        self.assertEqual(result2, AddReqResult.CONTINUE)

        # Fit last small prefill request
        req3 = self.create_mock_req("req3", priority=0, max_new_tokens=16)
        req3.extend_input_len = 3
        req3.host_hit_length = 0
        req3.prefix_indices = []
        req3.fill_ids = list(range(3))
        req3.last_node = MagicMock()
```
**EN:** This test exercises `test_mixed_chunk_prefill_budgets` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_chunk_prefill_budgets`。 这一段对应同一逻辑块的第 1 部分。

### Lines 435-443: test case mixed chunk prefill budgets (part 2/2) / 测试用例 mixed chunk prefill budgets（第 2/2 部分）
```python
        req3.sampling_params.ignore_eos = False

        result3 = adder2.add_one_req(
            req3, has_chunked_req=False, truncation_align_size=None
        )

        self.assertEqual(len(adder2.can_run_list), 2)
        self.assertEqual(adder2.rem_chunk_tokens, 0)  # 3 - 3 = 0
        self.assertEqual(result3, AddReqResult.OTHER)
```
**EN:** This test exercises `test_mixed_chunk_prefill_budgets` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_chunk_prefill_budgets`。 这一段对应同一逻辑块的第 2 部分。

### Lines 445-471: method build hybrid swa chunked req / 方法 build hybrid swa chunked req
```python
    def _build_hybrid_swa_chunked_req(
        self,
        *,
        page_size,
        rem_swa,
        rem_chunk=2048,
        extend_input_len=500,
        is_hybrid_swa=True,
        full_available=100_000,
    ):
        self.mock_token_allocator.swa_available_size.return_value = rem_swa
        self.mock_token_allocator.full_available_size.return_value = full_available
        self.mock_token_allocator.available_size.return_value = full_available
        self.mock_tree_cache.sliding_window_size = 128
        adder = self.create_adder(
            self.create_running_batch(),
            page_size=page_size,
            rem_chunk_tokens=rem_chunk,
        )
        adder.is_hybrid_swa = is_hybrid_swa

        req = self.create_mock_req("chunked", priority=0, max_new_tokens=128)
        req.extend_input_len = extend_input_len
        req.prefix_indices = []
        req.fill_ids = list(range(extend_input_len))
        req.set_extend_input_len = MagicMock()
        return adder, req
```
**EN:** This block implements `_build_hybrid_swa_chunked_req` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_hybrid_swa_chunked_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 473-490: test case add chunked req hybrid swa reserves page for alloc extend / 测试用例 add chunked req hybrid swa reserves page for alloc extend
```python
    def test_add_chunked_req_hybrid_swa_reserves_page_for_alloc_extend(self):
        # alloc_extend needs extend_num_tokens + page_size per request. If the
        # scheduler hands out all of rem_swa_tokens, alloc_extend cannot get its
        # extra page and OOMs. With the fix, extend_input_len must cap at
        # rem_swa_tokens - page_size so the page is reserved.
        PAGE_SIZE = 64
        REM_SWA = 100
        adder, req = self._build_hybrid_swa_chunked_req(
            page_size=PAGE_SIZE, rem_swa=REM_SWA
        )

        result = adder.add_chunked_req(req)

        self.assertIs(result, req)  # truncated → chunked prefill continues
        req.set_extend_input_len.assert_called_once()
        new_len = req.set_extend_input_len.call_args.args[0]
        self.assertLessEqual(new_len + PAGE_SIZE, REM_SWA)
        self.assertEqual(new_len, REM_SWA - PAGE_SIZE)
```
**EN:** This test exercises `test_add_chunked_req_hybrid_swa_reserves_page_for_alloc_extend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_chunked_req_hybrid_swa_reserves_page_for_alloc_extend`。

### Lines 492-507: test case add chunked req hybrid swa defers when swa below page / 测试用例 add chunked req hybrid swa defers when swa below page
```python
    def test_add_chunked_req_hybrid_swa_defers_when_swa_below_page(self):
        # When rem_swa_tokens <= page_size there is no room to serve even the
        # reservation, so the chunked req must be deferred (returned unchanged)
        # instead of falling back to rem_chunk_tokens and bypassing SWA budget.
        PAGE_SIZE = 64
        adder, req = self._build_hybrid_swa_chunked_req(
            page_size=PAGE_SIZE, rem_swa=PAGE_SIZE
        )
        original_len = req.extend_input_len

        result = adder.add_chunked_req(req)

        self.assertIs(result, req)
        req.set_extend_input_len.assert_not_called()
        self.assertEqual(req.extend_input_len, original_len)
        self.assertEqual(len(adder.can_run_list), 0)
```
**EN:** This test exercises `test_add_chunked_req_hybrid_swa_defers_when_swa_below_page` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_chunked_req_hybrid_swa_defers_when_swa_below_page`。

### Lines 509-527: test case swa budget for req / 测试用例 swa budget for req
```python
    def test_swa_budget_for_req(self):
        cases = [
            # (extend, rem_chunk, window, page, expected, label)
            (64, None, 128, 16, 128 + 16, "no_cap_floor_active"),
            (200, None, 256, 32, 256 + 32, "no_cap_floor_active_other_dims"),
            (300, None, 128, 16, 300 + 16, "no_cap_floor_inactive"),
            (200, 50, 64, 8, 64 + 8, "cap_binds_then_floor"),
            (300, 500, 64, 64, 300 + 64, "cap_does_not_bind"),
            (0, None, 128, 16, 128 + 16, "extend_zero_floor_only"),
        ]
        for extend, rem_chunk, window, page, expected, label in cases:
            with self.subTest(label=label):
                self.mock_tree_cache.sliding_window_size = window
                adder = self.create_adder(
                    self.create_running_batch(),
                    page_size=page,
                    rem_chunk_tokens=rem_chunk,
                )
                self.assertEqual(adder._swa_budget_for_req(extend), expected)
```
**EN:** This test exercises `test_swa_budget_for_req` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_swa_budget_for_req`。

### Lines 529-545: test case add chunked req non hybrid no swa reservation / 测试用例 add chunked req non hybrid no swa reservation
```python
    def test_add_chunked_req_non_hybrid_no_swa_reservation(self):
        # Non-hybrid path: the SWA-pool reservation must NOT apply, otherwise
        # the fix would regress non-SWA models.
        PAGE_SIZE = 16
        adder, req = self._build_hybrid_swa_chunked_req(
            page_size=PAGE_SIZE,
            rem_swa=10,
            rem_chunk=500,
            extend_input_len=200,
            is_hybrid_swa=False,
            full_available=300,
        )

        result = adder.add_chunked_req(req)
        self.assertIsNone(result)
        req.set_extend_input_len.assert_called_once_with(200)
        self.assertIn(req, adder.can_run_list)
```
**EN:** This test exercises `test_add_chunked_req_non_hybrid_no_swa_reservation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_chunked_req_non_hybrid_no_swa_reservation`。

### Lines 548-549: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPrefillAdder`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPrefillAdder.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPrefillAdder.create_tree_cache`: This helper encapsulates `create_tree_cache` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_tree_cache`，以便周围测试复用准备、执行或校验逻辑。
- `TestPrefillAdder.create_token_allocator`: This helper encapsulates `create_token_allocator` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_token_allocator`，以便周围测试复用准备、执行或校验逻辑。
- `TestPrefillAdder.create_running_batch`: This helper encapsulates `create_running_batch` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_running_batch`，以便周围测试复用准备、执行或校验逻辑。
- `TestPrefillAdder.create_server_args`: This helper encapsulates `create_server_args` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_server_args`，以便周围测试复用准备、执行或校验逻辑。
- `TestPrefillAdder.create_mock_req`: This helper encapsulates `create_mock_req` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_mock_req`，以便周围测试复用准备、执行或校验逻辑。
- `TestPrefillAdder.create_adder`: This helper encapsulates `create_adder` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_adder`，以便周围测试复用准备、执行或校验逻辑。
- `TestPrefillAdder.test_preempt_success_high_priority_values_first`: This test exercises `test_preempt_success_high_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_success_high_priority_values_first`。
- `TestPrefillAdder.test_preempt_success_low_priority_values_first`: This test exercises `test_preempt_success_low_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_success_low_priority_values_first`。
- `TestPrefillAdder.test_preempt_fail_low_priority_values_first`: This test exercises `test_preempt_fail_low_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_fail_low_priority_values_first`。
- `TestPrefillAdder.test_preempt_fail_high_priority_values_first`: This test exercises `test_preempt_fail_high_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preempt_fail_high_priority_values_first`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.schedule_policy`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.server_args`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 549
