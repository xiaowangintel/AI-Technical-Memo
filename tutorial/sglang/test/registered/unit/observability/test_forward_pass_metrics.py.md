# test_forward_pass_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/observability/test_forward_pass_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates forward pass metrics behavior in SGLang's unit / observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / observability 领域中与 forward pass metrics 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-3: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 5-14: module imports and dependencies / 模块导入与依赖
```python
import types
import unittest
from unittest.mock import patch

from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.distributed.parallel_state_wrapper import ParallelState
from sglang.srt.managers.scheduler_components.metrics_reporter import (
    PrefillStats,
    SchedulerMetricsReporter,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `types`, `unittest`, `unittest.mock`, `sglang.srt.disaggregation.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `types`, `unittest`, `unittest.mock`, `sglang.srt.disaggregation.utils`。

### Lines 17-39: function make ps / 函数 make ps
```python
def _make_ps(**overrides) -> ParallelState:
    """Build a ParallelState with reasonable defaults for tests; override fields via kwargs."""
    defaults = dict(
        tp_rank=0,
        tp_size=1,
        pp_rank=0,
        pp_size=1,
        dp_rank=None,
        dp_size=1,
        attn_tp_rank=0,
        attn_tp_size=1,
        attn_cp_rank=0,
        attn_cp_size=1,
        attn_dp_rank=0,
        attn_dp_size=1,
        moe_ep_rank=0,
        moe_ep_size=1,
        moe_dp_rank=None,
        moe_dp_size=1,
        gpu_id=0,
    )
    defaults.update(overrides)
    return ParallelState(**defaults)
```
**EN:** Build a ParallelState with reasonable defaults for tests; override fields via kwargs. This block implements `_make_ps` and captures one focused piece of the module's behavior.
**CN:** Build a ParallelState with reasonable defaults for tests; override fields via kwargs. 该代码块实现 `_make_ps`，承担模块行为中的一个聚焦逻辑片段。

### Lines 42-42: class _FakeReq declaration / 类 _FakeReq 声明
```python
class _FakeReq:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 43-52: method init / 方法 init
```python
    def __init__(
        self,
        prompt_len: int,
        output_len: int = 0,
        prefix_len: int = 0,
    ):
        self.origin_input_ids = list(range(prompt_len))
        self.output_ids = list(range(output_len))
        self.prefix_indices = list(range(prefix_len))
        self.seqlen = prompt_len + output_len
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 55-55: class _FakeForwardMode declaration / 类 _FakeForwardMode 声明
```python
class _FakeForwardMode:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 56-58: method init / 方法 init
```python
    def __init__(self, *, is_mixed: bool = False, is_extend: bool = False):
        self._is_mixed = is_mixed
        self._is_extend = is_extend
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 60-61: method is mixed / 方法 is mixed
```python
    def is_mixed(self):
        return self._is_mixed
```
**EN:** This block implements `is_mixed` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `is_mixed`，承担模块行为中的一个聚焦逻辑片段。

### Lines 63-64: method is extend / 方法 is extend
```python
    def is_extend(self, include_draft_extend_v2: bool = False):
        return self._is_extend
```
**EN:** This block implements `is_extend` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `is_extend`，承担模块行为中的一个聚焦逻辑片段。

### Lines 66-67: method is decode / 方法 is decode
```python
    def is_decode(self):
        return not self._is_mixed and not self._is_extend
```
**EN:** This block implements `is_decode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `is_decode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 70-70: class _CollectingPublisher declaration / 类 _CollectingPublisher 声明
```python
class _CollectingPublisher:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 71-72: method init / 方法 init
```python
    def __init__(self):
        self.metrics = []
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 74-75: method publish / 方法 publish
```python
    def publish(self, metrics):
        self.metrics.append(metrics)
```
**EN:** This block implements `publish` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `publish`，承担模块行为中的一个聚焦逻辑片段。

### Lines 78-78: class _DummyPublisherThread declaration / 类 _DummyPublisherThread 声明
```python
class _DummyPublisherThread:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 79-82: method init / 方法 init
```python
    def __init__(self, endpoint: str, worker_id: str, dp_rank: int, **_: object):
        self.endpoint = endpoint
        self.worker_id = worker_id
        self.dp_rank = dp_rank
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 84-85: method shutdown / 方法 shutdown
```python
    def shutdown(self):
        pass
```
**EN:** This block implements `shutdown` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `shutdown`，承担模块行为中的一个聚焦逻辑片段。

### Lines 88-125: function make reporter / 函数 make reporter
```python
def _make_reporter(scheduler) -> SchedulerMetricsReporter:
    if not hasattr(scheduler, "server_args"):
        scheduler.server_args = types.SimpleNamespace(
            enable_metrics=False,
            enable_metrics_for_all_schedulers=False,
            kv_events_config=None,
            enable_mfu_metrics=False,
            enable_forward_pass_metrics=False,
        )
    if not hasattr(scheduler, "ps"):
        scheduler.ps = types.SimpleNamespace(attn_tp_rank=0, attn_cp_rank=0)
    if not hasattr(scheduler, "kv_events_publisher"):
        scheduler.kv_events_publisher = types.SimpleNamespace(
            init_kv_events=lambda *a, **kw: None,
        )
    if not hasattr(scheduler, "tp_workers"):
        scheduler.tp_workers = []
    if not hasattr(scheduler, "tp_worker"):
        scheduler.tp_worker = types.SimpleNamespace(
            model_runner=types.SimpleNamespace(),
        )
    if not hasattr(scheduler, "draft_worker"):
        scheduler.draft_worker = None
    context = types.SimpleNamespace(
        enable_metrics=False,
        is_stats_logging_rank=True,
        current_scheduler_metrics_enabled=False,
        enable_kv_cache_events=False,
        collector=None,
    )
    return SchedulerMetricsReporter(
        scheduler=scheduler,
        tp_rank=0,
        pp_rank=0,
        dp_rank=0,
        metrics_collector_context=context,
        metrics_collector=None,
    )
```
**EN:** This block implements `_make_reporter` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_reporter`，承担模块行为中的一个聚焦逻辑片段。

### Lines 128-128: class TestForwardPassMetrics declaration / 类 TestForwardPassMetrics 声明
```python
class TestForwardPassMetrics(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 129-139: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.scheduler = types.SimpleNamespace()
        self.scheduler._fpm_worker_id = "worker-7"
        self.scheduler._fpm_dp_rank = 0
        self.scheduler._fpm_publisher = _CollectingPublisher()
        self.scheduler._fpm_uses_device_timer = False
        self.scheduler._fpm_gpu_time_acc = 0.0
        self.scheduler.waiting_queue = []
        self.scheduler.disaggregation_mode = DisaggregationMode.NULL
        self.reporter = _make_reporter(self.scheduler)
        self.scheduler.enable_fpm = True
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 141-151: method make batch / 方法 make batch
```python
    def _make_batch(self, **overrides):
        defaults = dict(
            forward_mode=_FakeForwardMode(),
            reqs=[],
            decoding_reqs=[],
            prefill_stats=None,
            seq_lens_cpu=[],
            fpm_start_time=100.0,
        )
        defaults.update(overrides)
        return types.SimpleNamespace(**defaults)
```
**EN:** This block implements `_make_batch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_batch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 153-193: test case emit mixed batch separates prefill and decode / 测试用例 emit mixed batch separates prefill and decode
```python
    def test_emit_mixed_batch_separates_prefill_and_decode(self):
        self.scheduler._fpm_dp_rank = 3
        self.scheduler.waiting_queue = [_FakeReq(6), _FakeReq(4, output_len=2)]

        prefill_a = _FakeReq(10, prefix_len=2)
        prefill_b = _FakeReq(14, prefix_len=3)
        decode_req = _FakeReq(8, output_len=3)
        batch = self._make_batch(
            forward_mode=_FakeForwardMode(is_mixed=True, is_extend=True),
            reqs=[prefill_a, prefill_b, decode_req],
            decoding_reqs=[decode_req],
            prefill_stats=PrefillStats(
                log_input_tokens=12,
                log_hit_tokens=5,
                new_token_ratio=1.0,
                num_running_reqs=types.SimpleNamespace(),
                num_new_seqs=2,
            ),
            seq_lens_cpu=[decode_req.seqlen],
        )

        with patch(
            "sglang.srt.managers.scheduler_components.metrics_reporter.time.monotonic",
            return_value=104.5,
        ):
            self.reporter._emit_forward_pass_metrics(batch)

        self.assertEqual(len(self.scheduler._fpm_publisher.metrics), 1)
        metrics = self.scheduler._fpm_publisher.metrics[0]
        self.assertEqual(metrics.worker_id, "worker-7")
        self.assertEqual(metrics.dp_rank, 3)
        self.assertEqual(metrics.wall_time, 4.5)
        self.assertEqual(metrics.scheduled_requests.num_prefill_requests, 2)
        self.assertEqual(metrics.scheduled_requests.sum_prefill_tokens, 12)
        self.assertEqual(metrics.scheduled_requests.sum_prefill_kv_tokens, 5)
        self.assertEqual(metrics.scheduled_requests.num_decode_requests, 1)
        self.assertEqual(
            metrics.scheduled_requests.sum_decode_kv_tokens, decode_req.seqlen
        )
        self.assertEqual(metrics.queued_requests.num_prefill_requests, 1)
        self.assertEqual(metrics.queued_requests.num_decode_requests, 1)
```
**EN:** This test exercises `test_emit_mixed_batch_separates_prefill_and_decode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_emit_mixed_batch_separates_prefill_and_decode`。

### Lines 195-209: test case emit uses device timer gpu time / 测试用例 emit uses device timer gpu time
```python
    def test_emit_uses_device_timer_gpu_time(self):
        self.scheduler._fpm_uses_device_timer = True
        self.scheduler._fpm_gpu_time_acc = 0.042
        self.reporter.forward_pass_device_timer = types.SimpleNamespace(
            _report=lambda: None,
        )
        batch = self._make_batch()

        self.reporter._emit_forward_pass_metrics(batch)

        self.assertEqual(len(self.scheduler._fpm_publisher.metrics), 1)
        self.assertAlmostEqual(
            self.scheduler._fpm_publisher.metrics[0].wall_time, 0.042, places=4
        )
        self.assertAlmostEqual(self.scheduler._fpm_gpu_time_acc, 0.0)
```
**EN:** This test exercises `test_emit_uses_device_timer_gpu_time` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_emit_uses_device_timer_gpu_time`。

### Lines 211-221: test case emit skips when device timer zero / 测试用例 emit skips when device timer zero
```python
    def test_emit_skips_when_device_timer_zero(self):
        self.scheduler._fpm_uses_device_timer = True
        self.scheduler._fpm_gpu_time_acc = 0.0
        self.reporter.forward_pass_device_timer = types.SimpleNamespace(
            _report=lambda: None,
        )
        batch = self._make_batch()

        self.reporter._emit_forward_pass_metrics(batch)

        self.assertEqual(len(self.scheduler._fpm_publisher.metrics), 0)
```
**EN:** This test exercises `test_emit_skips_when_device_timer_zero` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_emit_skips_when_device_timer_zero`。

### Lines 223-235: test case emit uses monotonic without device timer / 测试用例 emit uses monotonic without device timer
```python
    def test_emit_uses_monotonic_without_device_timer(self):
        batch = self._make_batch()

        with patch(
            "sglang.srt.managers.scheduler_components.metrics_reporter.time.monotonic",
            return_value=100.035,
        ):
            self.reporter._emit_forward_pass_metrics(batch, result=None)

        self.assertEqual(len(self.scheduler._fpm_publisher.metrics), 1)
        self.assertAlmostEqual(
            self.scheduler._fpm_publisher.metrics[0].wall_time, 0.035, places=4
        )
```
**EN:** This test exercises `test_emit_uses_monotonic_without_device_timer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_emit_uses_monotonic_without_device_timer`。

### Lines 237-253: test case disagg prefill queued metrics / 测试用例 disagg prefill queued metrics
```python
    def test_disagg_prefill_queued_metrics(self):
        self.scheduler.disaggregation_mode = DisaggregationMode.PREFILL
        self.scheduler.disagg_prefill_bootstrap_queue = types.SimpleNamespace(
            queue=[_FakeReq(100), _FakeReq(200), _FakeReq(50)],
        )
        batch = self._make_batch()

        with patch(
            "sglang.srt.managers.scheduler_components.metrics_reporter.time.monotonic",
            return_value=101.0,
        ):
            self.reporter._emit_forward_pass_metrics(batch)

        metrics = self.scheduler._fpm_publisher.metrics[0]
        self.assertEqual(metrics.queued_requests.num_prefill_requests, 3)
        self.assertEqual(metrics.queued_requests.sum_prefill_tokens, 350)
        self.assertEqual(metrics.queued_requests.num_decode_requests, 0)
```
**EN:** This test exercises `test_disagg_prefill_queued_metrics` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disagg_prefill_queued_metrics`。

### Lines 255-274: test case disagg decode queued metrics / 测试用例 disagg decode queued metrics
```python
    def test_disagg_decode_queued_metrics(self):
        self.scheduler.disaggregation_mode = DisaggregationMode.DECODE
        self.scheduler.disagg_decode_prealloc_queue = types.SimpleNamespace(
            queue=[_FakeReq(10, output_len=5), _FakeReq(20, output_len=10)],
        )
        self.scheduler.disagg_decode_transfer_queue = types.SimpleNamespace(
            queue=[_FakeReq(30, output_len=15)],
        )
        batch = self._make_batch()

        with patch(
            "sglang.srt.managers.scheduler_components.metrics_reporter.time.monotonic",
            return_value=101.0,
        ):
            self.reporter._emit_forward_pass_metrics(batch)

        metrics = self.scheduler._fpm_publisher.metrics[0]
        self.assertEqual(metrics.queued_requests.num_prefill_requests, 0)
        self.assertEqual(metrics.queued_requests.num_decode_requests, 3)
        self.assertEqual(metrics.queued_requests.sum_decode_kv_tokens, 15 + 30 + 45)
```
**EN:** This test exercises `test_disagg_decode_queued_metrics` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disagg_decode_queued_metrics`。

### Lines 276-302: test case init metrics uses server worker id / 测试用例 init metrics uses server worker id
```python
    def test_init_metrics_uses_server_worker_id(self):
        scheduler = types.SimpleNamespace()
        scheduler.server_args = types.SimpleNamespace(
            enable_metrics=False,
            enable_metrics_for_all_schedulers=False,
            extra_metric_labels=None,
            enable_forward_pass_metrics=True,
            forward_pass_metrics_worker_id="endpoint-42",
            forward_pass_metrics_ipc_name=None,
            kv_events_config=None,
        )
        scheduler.ps = _make_ps(attn_tp_rank=0, dp_rank=2, pp_rank=0, pp_size=1)
        scheduler.enable_kv_cache_events = False

        with patch(
            "sglang.srt.observability.forward_pass_metrics._FpmPublisherThread",
            _DummyPublisherThread,
        ):
            reporter = _make_reporter(scheduler)

        self.assertTrue(scheduler.enable_fpm)
        self.assertEqual(scheduler._fpm_worker_id, "endpoint-42")
        self.assertEqual(scheduler._fpm_dp_rank, 2)
        self.assertEqual(scheduler._fpm_publisher.worker_id, "endpoint-42")
        self.assertEqual(scheduler._fpm_publisher.dp_rank, 2)
        self.assertTrue(scheduler._fpm_publisher.endpoint.startswith("ipc://"))
        self.assertIsNotNone(scheduler.server_args.forward_pass_metrics_ipc_name)
```
**EN:** This test exercises `test_init_metrics_uses_server_worker_id` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_metrics_uses_server_worker_id`。

### Lines 304-324: test case init fpm disabled on non last pp rank / 测试用例 init fpm disabled on non last pp rank
```python
    def test_init_fpm_disabled_on_non_last_pp_rank(self):
        scheduler = types.SimpleNamespace()
        scheduler.server_args = types.SimpleNamespace(
            enable_metrics=False,
            enable_metrics_for_all_schedulers=False,
            extra_metric_labels=None,
            enable_forward_pass_metrics=True,
            forward_pass_metrics_worker_id="endpoint-42",
            forward_pass_metrics_ipc_name=None,
            kv_events_config=None,
        )
        scheduler.ps = _make_ps(attn_tp_rank=0, dp_rank=0, pp_rank=0, pp_size=2)
        scheduler.enable_kv_cache_events = False

        with patch(
            "sglang.srt.observability.forward_pass_metrics._FpmPublisherThread",
            _DummyPublisherThread,
        ):
            reporter = _make_reporter(scheduler)

        self.assertFalse(scheduler.enable_fpm)
```
**EN:** This test exercises `test_init_fpm_disabled_on_non_last_pp_rank` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_fpm_disabled_on_non_last_pp_rank`。

### Lines 327-328: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_ps`: Build a ParallelState with reasonable defaults for tests; override fields via kwargs. / 该代码块实现 `_make_ps`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeReq`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_FakeForwardMode`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_CollectingPublisher`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_DummyPublisherThread`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_make_reporter`: This block implements `_make_reporter` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_reporter`，承担模块行为中的一个聚焦逻辑片段。
- `TestForwardPassMetrics`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_FakeReq.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeForwardMode.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeForwardMode.is_mixed`: This block implements `is_mixed` and captures one focused piece of the module's behavior. / 该代码块实现 `is_mixed`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeForwardMode.is_extend`: This block implements `is_extend` and captures one focused piece of the module's behavior. / 该代码块实现 `is_extend`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeForwardMode.is_decode`: This block implements `is_decode` and captures one focused piece of the module's behavior. / 该代码块实现 `is_decode`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `types`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.disaggregation.utils`, `sglang.srt.distributed.parallel_state_wrapper`, `sglang.srt.managers.scheduler_components.metrics_reporter`

- **Total lines / 总行数**: 328
