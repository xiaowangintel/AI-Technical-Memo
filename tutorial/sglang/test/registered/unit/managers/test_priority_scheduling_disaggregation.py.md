# test_priority_scheduling_disaggregation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_priority_scheduling_disaggregation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates priority scheduling disaggregation behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 priority scheduling disaggregation 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and dependencies / 模块导入与依赖
```python
import sys
import unittest
from types import SimpleNamespace
from unittest.mock import MagicMock, patch

import torch

from sglang.srt.disaggregation.decode import (  # noqa: E402
    DecodePreallocQueue,
    SchedulerDisaggregationDecodeMixin,
)
from sglang.srt.disaggregation.utils import DisaggregationMode  # noqa: E402
from sglang.srt.managers.schedule_batch import FINISH_ABORT  # noqa: E402
from sglang.srt.managers.scheduler import Scheduler  # noqa: E402
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sys`, `unittest`, `types`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sys`, `unittest`, `types`, `unittest.mock`。

### Lines 17-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=5, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class TestDisaggregationPriorityQueueing declaration / 类 TestDisaggregationPriorityQueueing 声明
```python
class TestDisaggregationPriorityQueueing(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 21-34: method new scheduler / 方法 new scheduler
```python
    def _new_scheduler(self, disaggregation_mode: DisaggregationMode) -> Scheduler:
        scheduler = Scheduler.__new__(Scheduler)
        scheduler.disaggregation_mode = disaggregation_mode
        scheduler.enable_priority_scheduling = True
        scheduler.schedule_low_priority_values_first = False
        scheduler.abort_on_priority_when_disabled = False
        scheduler.waiting_queue = []
        scheduler._prefetch_kvcache = MagicMock()
        scheduler._abort_on_queued_limit = MagicMock(return_value=False)
        scheduler.model_config = SimpleNamespace(num_key_value_heads=8)
        scheduler.disagg_prefill_bootstrap_queue = MagicMock()
        scheduler.disagg_decode_prealloc_queue = MagicMock()
        scheduler.send_to_tokenizer = MagicMock()
        return scheduler
```
**EN:** This block implements `_new_scheduler` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_new_scheduler`，承担模块行为中的一个聚焦逻辑片段。

### Lines 36-42: method new req / 方法 new req
```python
    def _new_req(self, priority=None):
        req = MagicMock()
        req.priority = priority
        req.rid = "req"
        req.time_stats = MagicMock()
        req.time_stats.trace_ctx = MagicMock()
        return req
```
**EN:** This block implements `_new_req` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_new_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 44-52: test case prefill mode assigns default priority before bootstrap queue / 测试用例 prefill mode assigns default priority before bootstrap queue
```python
    def test_prefill_mode_assigns_default_priority_before_bootstrap_queue(self):
        scheduler = self._new_scheduler(DisaggregationMode.PREFILL)
        req = self._new_req(priority=None)

        scheduler._add_request_to_queue(req)

        self.assertEqual(req.priority, -sys.maxsize - 1)
        scheduler.disagg_prefill_bootstrap_queue.add.assert_called_once_with(req, 8)
        req.time_stats.set_prefill_bootstrap_queue_entry_time.assert_called_once()
```
**EN:** This test exercises `test_prefill_mode_assigns_default_priority_before_bootstrap_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prefill_mode_assigns_default_priority_before_bootstrap_queue`。

### Lines 54-64: test case decode mode assigns default priority before prealloc queue / 测试用例 decode mode assigns default priority before prealloc queue
```python
    def test_decode_mode_assigns_default_priority_before_prealloc_queue(self):
        scheduler = self._new_scheduler(DisaggregationMode.DECODE)
        req = self._new_req(priority=None)

        scheduler._add_request_to_queue(req)

        self.assertEqual(req.priority, -sys.maxsize - 1)
        scheduler.disagg_decode_prealloc_queue.add.assert_called_once_with(
            req, is_retracted=False
        )
        req.time_stats.set_decode_prealloc_queue_entry_time.assert_called_once()
```
**EN:** This test exercises `test_decode_mode_assigns_default_priority_before_prealloc_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_mode_assigns_default_priority_before_prealloc_queue`。

### Lines 66-76: test case priority disabled abort validation applies to decode mode / 测试用例 priority disabled abort validation applies to decode mode
```python
    def test_priority_disabled_abort_validation_applies_to_decode_mode(self):
        scheduler = self._new_scheduler(DisaggregationMode.DECODE)
        scheduler.enable_priority_scheduling = False
        scheduler.abort_on_priority_when_disabled = True
        req = self._new_req(priority=10)

        scheduler._add_request_to_queue(req)

        scheduler.disagg_decode_prealloc_queue.add.assert_not_called()
        scheduler.send_to_tokenizer.send_output.assert_called_once()
        req.time_stats.trace_ctx.abort.assert_called_once()
```
**EN:** This test exercises `test_priority_disabled_abort_validation_applies_to_decode_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_disabled_abort_validation_applies_to_decode_mode`。

### Lines 79-79: class TestDecodePreallocQueuePriority declaration / 类 TestDecodePreallocQueuePriority 声明
```python
class TestDecodePreallocQueuePriority(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 80-98: method new decode req / 方法 new decode req
```python
    def _new_decode_req(self, rid: str, priority: int, *, failed: bool = False):
        req = SimpleNamespace(
            rid=rid,
            priority=priority,
            origin_input_ids=[1, 2, 3],
            output_ids=[],
            req_pool_idx=int(priority) % 8,
            finished_reason=FINISH_ABORT("failed") if failed else None,
            return_logprob=False,
            sampling_params=SimpleNamespace(max_new_tokens=8),
            cache_protected_len=0,
            time_stats=MagicMock(),
        )
        return SimpleNamespace(
            req=req,
            waiting_for_input=True,
            kv_receiver=MagicMock(),
            metadata_buffer_index=-1,
        )
```
**EN:** This block implements `_new_decode_req` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_new_decode_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 100-143: method new queue / 方法 new queue
```python
    def _new_queue(self, decode_reqs, *, low_priority_values_first: bool = False):
        queue = DecodePreallocQueue.__new__(DecodePreallocQueue)
        queue.queue = list(decode_reqs)
        queue.pending_reqs = []
        queue.retracted_queue = []
        queue.num_reserved_decode_tokens = 0
        queue._resolve_pending_reqs = MagicMock()
        queue._update_handshake_waiters = MagicMock()
        queue._allocatable_tokens = MagicMock(return_value=1000)
        queue._pre_alloc = MagicMock(
            side_effect=lambda req, prefix_indices=None, prefix_len=0: torch.arange(
                len(req.origin_input_ids) - prefix_len, dtype=torch.int64
            )
        )

        queue.req_to_token_pool = MagicMock()
        queue.req_to_token_pool.available_size.return_value = 100
        queue.req_to_token_pool.req_to_token = torch.arange(
            8 * 16, dtype=torch.int64
        ).reshape(8, 16)

        queue.req_to_metadata_buffer_idx_allocator = MagicMock()
        queue.req_to_metadata_buffer_idx_allocator.available_size.return_value = 100
        queue.req_to_metadata_buffer_idx_allocator.alloc.side_effect = iter(range(100))

        queue.token_to_kv_pool_allocator = MagicMock()
        queue.token_to_kv_pool_allocator.page_size = 1
        queue.token_to_kv_pool_allocator.available_size.return_value = 1000
        queue.token_to_kv_pool = MagicMock()
        queue.transfer_queue = SimpleNamespace(queue=[], enable_staging=False)
        queue.kv_manager = SimpleNamespace(kv_args=SimpleNamespace(state_types=[]))
        queue.tree_cache = MagicMock()

        scheduler = MagicMock()
        scheduler.enable_priority_scheduling = True
        scheduler.schedule_low_priority_values_first = low_priority_values_first
        scheduler.running_batch.reqs = []
        scheduler.server_args.disaggregation_decode_enable_radix_cache = False
        scheduler.enable_hisparse = False
        scheduler.waiting_queue = []
        scheduler.last_batch = None
        scheduler.output_streamer = MagicMock()
        queue.scheduler = scheduler
        return queue
```
**EN:** This block implements `_new_queue` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_new_queue`，承担模块行为中的一个聚焦逻辑片段。

### Lines 145-164: test case prealloc queue schedules higher priority values first by default / 测试用例 prealloc queue schedules higher priority values first by default
```python
    def test_prealloc_queue_schedules_higher_priority_values_first_by_default(self):
        reqs = [
            self._new_decode_req("low", 1),
            self._new_decode_req("high", 10),
            self._new_decode_req("mid", 5),
        ]
        queue = self._new_queue(reqs)

        with patch("sglang.srt.disaggregation.decode.CLIP_MAX_NEW_TOKEN", 4096):
            preallocated, failed = queue.pop_preallocated()

        self.assertEqual(
            [decode_req.req.rid for decode_req in preallocated],
            [
                "high",
                "mid",
                "low",
            ],
        )
        self.assertEqual(failed, [])
```
**EN:** This test exercises `test_prealloc_queue_schedules_higher_priority_values_first_by_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prealloc_queue_schedules_higher_priority_values_first_by_default`。

### Lines 166-185: test case prealloc queue can schedule lower priority values first / 测试用例 prealloc queue can schedule lower priority values first
```python
    def test_prealloc_queue_can_schedule_lower_priority_values_first(self):
        reqs = [
            self._new_decode_req("mid", 5),
            self._new_decode_req("high", 10),
            self._new_decode_req("low", 1),
        ]
        queue = self._new_queue(reqs, low_priority_values_first=True)

        with patch("sglang.srt.disaggregation.decode.CLIP_MAX_NEW_TOKEN", 4096):
            preallocated, failed = queue.pop_preallocated()

        self.assertEqual(
            [decode_req.req.rid for decode_req in preallocated],
            [
                "low",
                "mid",
                "high",
            ],
        )
        self.assertEqual(failed, [])
```
**EN:** This test exercises `test_prealloc_queue_can_schedule_lower_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prealloc_queue_can_schedule_lower_priority_values_first`。

### Lines 187-202: test case failed request indices stay valid after priority sort / 测试用例 failed request indices stay valid after priority sort
```python
    def test_failed_request_indices_stay_valid_after_priority_sort(self):
        failed_low = self._new_decode_req("failed-low", 1, failed=True)
        healthy_high = self._new_decode_req("healthy-high", 10)
        queue = self._new_queue([failed_low, healthy_high])

        with patch("sglang.srt.disaggregation.decode.CLIP_MAX_NEW_TOKEN", 4096):
            preallocated, failed = queue.pop_preallocated()

        self.assertEqual(
            [decode_req.req.rid for decode_req in preallocated], ["healthy-high"]
        )
        self.assertEqual([decode_req.req.rid for decode_req in failed], ["failed-low"])
        self.assertEqual(queue.queue, [])
        queue.scheduler.output_streamer.stream_output.assert_called_once_with(
            [failed_low.req], failed_low.req.return_logprob
        )
```
**EN:** This test exercises `test_failed_request_indices_stay_valid_after_priority_sort` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_failed_request_indices_stay_valid_after_priority_sort`。

### Lines 205-205: class TestDecodePrebuiltPriority declaration / 类 TestDecodePrebuiltPriority 声明
```python
class TestDecodePrebuiltPriority(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 206-246: test case waiting queue is sorted before prebuilt selection / 测试用例 waiting queue is sorted before prebuilt selection
```python
    def test_waiting_queue_is_sorted_before_prebuilt_selection(self):
        scheduler = Scheduler.__new__(Scheduler)
        scheduler.grammar_manager = MagicMock()
        scheduler.grammar_manager.has_waiting_grammars.return_value = False
        original_waiting_queue = [MagicMock(rid="low"), MagicMock(rid="high")]
        scheduler.waiting_queue = original_waiting_queue
        scheduler.waiting_queue[0].priority = 1
        scheduler.waiting_queue[1].priority = 10
        scheduler.enable_priority_scheduling = True
        scheduler.running_batch = MagicMock()
        scheduler.running_batch.batch_size.return_value = 0
        scheduler.req_to_token_pool = MagicMock(size=1)
        scheduler.token_to_kv_pool_allocator = MagicMock()
        scheduler.tree_cache = MagicMock()
        scheduler.model_config = MagicMock()
        scheduler.enable_overlap = False
        scheduler.spec_algorithm = MagicMock()
        scheduler.max_running_requests = 1
        scheduler.server_args = SimpleNamespace(
            disaggregation_decode_enable_radix_cache=False
        )
        scheduler.future_map = MagicMock()
        scheduler.policy = MagicMock()
        scheduler.policy.calc_priority.side_effect = (
            lambda waiting_queue, _: waiting_queue.sort(key=lambda req: -req.priority)
        )

        new_batch = MagicMock()
        with patch(
            "sglang.srt.disaggregation.decode.ScheduleBatch.init_new",
            return_value=new_batch,
        ) as init_new:
            ret = SchedulerDisaggregationDecodeMixin.get_new_prebuilt_batch(scheduler)

        self.assertIs(ret, new_batch)
        scheduler.policy.calc_priority.assert_called_once_with(
            original_waiting_queue, scheduler.running_batch
        )
        selected_reqs = init_new.call_args.args[0]
        self.assertEqual([req.rid for req in selected_reqs], ["high"])
        self.assertEqual([req.rid for req in scheduler.waiting_queue], ["low"])
```
**EN:** This test exercises `test_waiting_queue_is_sorted_before_prebuilt_selection` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_waiting_queue_is_sorted_before_prebuilt_selection`。

### Lines 249-250: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDisaggregationPriorityQueueing`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDecodePreallocQueuePriority`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDecodePrebuiltPriority`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDisaggregationPriorityQueueing._new_scheduler`: This block implements `_new_scheduler` and captures one focused piece of the module's behavior. / 该代码块实现 `_new_scheduler`，承担模块行为中的一个聚焦逻辑片段。
- `TestDisaggregationPriorityQueueing._new_req`: This block implements `_new_req` and captures one focused piece of the module's behavior. / 该代码块实现 `_new_req`，承担模块行为中的一个聚焦逻辑片段。
- `TestDisaggregationPriorityQueueing.test_prefill_mode_assigns_default_priority_before_bootstrap_queue`: This test exercises `test_prefill_mode_assigns_default_priority_before_bootstrap_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prefill_mode_assigns_default_priority_before_bootstrap_queue`。
- `TestDisaggregationPriorityQueueing.test_decode_mode_assigns_default_priority_before_prealloc_queue`: This test exercises `test_decode_mode_assigns_default_priority_before_prealloc_queue` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_mode_assigns_default_priority_before_prealloc_queue`。
- `TestDisaggregationPriorityQueueing.test_priority_disabled_abort_validation_applies_to_decode_mode`: This test exercises `test_priority_disabled_abort_validation_applies_to_decode_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_disabled_abort_validation_applies_to_decode_mode`。
- `TestDecodePreallocQueuePriority._new_decode_req`: This block implements `_new_decode_req` and captures one focused piece of the module's behavior. / 该代码块实现 `_new_decode_req`，承担模块行为中的一个聚焦逻辑片段。
- `TestDecodePreallocQueuePriority._new_queue`: This block implements `_new_queue` and captures one focused piece of the module's behavior. / 该代码块实现 `_new_queue`，承担模块行为中的一个聚焦逻辑片段。
- `TestDecodePreallocQueuePriority.test_prealloc_queue_schedules_higher_priority_values_first_by_default`: This test exercises `test_prealloc_queue_schedules_higher_priority_values_first_by_default` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prealloc_queue_schedules_higher_priority_values_first_by_default`。
- `TestDecodePreallocQueuePriority.test_prealloc_queue_can_schedule_lower_priority_values_first`: This test exercises `test_prealloc_queue_can_schedule_lower_priority_values_first` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prealloc_queue_can_schedule_lower_priority_values_first`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `sys`, `unittest`, `types`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.decode`, `sglang.srt.disaggregation.utils`, `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.scheduler`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 250
