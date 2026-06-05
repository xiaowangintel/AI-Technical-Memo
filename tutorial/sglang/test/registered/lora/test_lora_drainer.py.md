# test_lora_drainer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_drainer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora drainer behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora drainer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from typing import cast
from unittest import mock

from sglang.srt.lora.lora_drainer import LoRADrainer
from sglang.srt.managers.schedule_batch import Req
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.lora_utils import (
    CI_MULTI_LORA_MODELS,
    run_lora_batch_splitting_equivalence_test,
)
from sglang.test.test_utils import is_in_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `typing`, `sglang.srt.lora.lora_drainer`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `typing`, `sglang.srt.lora.lora_drainer`。

### Lines 15-19: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=100, stage="extra-a", runner_config="1-gpu-small")
register_amd_ci(est_time=100, suite="stage-b-test-1-gpu-small-amd")

MOCK_START_TIME = 1000.0
LORA_DRAIN_WAIT_THRESHOLD = 3.0
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 22-31: helper routine make req / 辅助流程 make req
```python
def make_req(lora_id, wait_queue_entry_time, max_new_tokens, output_len=0):
    time_stats = SimpleNamespace(wait_queue_entry_time=wait_queue_entry_time)
    sampling_params = SimpleNamespace(max_new_tokens=max_new_tokens)
    req_ns = SimpleNamespace(
        lora_id=lora_id,
        time_stats=time_stats,
        sampling_params=sampling_params,
        output_ids=[0] * output_len,
    )
    return cast(Req, req_ns)
```
**EN:** This helper encapsulates `make_req` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `make_req`，以便周围测试复用准备、执行或校验逻辑。

### Lines 34-34: class TestLoRADrainer declaration / 类 TestLoRADrainer 声明
```python
class TestLoRADrainer(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 35-86: test case update draining marks adapter / 测试用例 update draining marks adapter
```python
    def test_update_draining_marks_adapter(self):
        if is_in_ci():
            return

        with mock.patch("time.monotonic", return_value=MOCK_START_TIME):
            drainer = LoRADrainer(
                max_loras_per_batch=1, max_wait_time_secs=LORA_DRAIN_WAIT_THRESHOLD
            )

            # Waiting request for adapter 'A' that has been waiting longer than threshold
            wait_entry = MOCK_START_TIME - (LORA_DRAIN_WAIT_THRESHOLD + 0.01)
            waiting_req = make_req("A", wait_entry, max_new_tokens=10)

            running_req = make_req("B", wait_entry, max_new_tokens=100, output_len=0)

            drainer.update_draining_state(
                waiting_queue=[waiting_req],
                running_reqs=[running_req],
            )

            # Running adapter 'B' should be marked as draining for 'A'
            self.assertEqual(drainer.adapter_to_stats["B"].is_draining_for, "A")

            # Once running adapter 'B' finishes running, it should no longer be draining
            drainer.update_draining_state(waiting_queue=[waiting_req], running_reqs=[])
            self.assertIsNone(drainer.adapter_to_stats["B"].is_draining_for)

        with mock.patch("time.monotonic", return_value=MOCK_START_TIME):
            drainer = LoRADrainer(
                max_loras_per_batch=2, max_wait_time_secs=LORA_DRAIN_WAIT_THRESHOLD
            )

            # Two starving adapters should cause two running adapters to drain.
            wait_entryA = MOCK_START_TIME - (LORA_DRAIN_WAIT_THRESHOLD + 0.05)
            wait_entryD = MOCK_START_TIME - (LORA_DRAIN_WAIT_THRESHOLD + 0.01)
            starving_a = make_req("A", wait_entryA, max_new_tokens=10)
            starving_d = make_req("D", wait_entryD, max_new_tokens=10)

            # Running adapters B and C with different remaining tokens
            running_b = make_req("B", wait_entryA, max_new_tokens=5, output_len=0)
            running_c = make_req("C", wait_entryA, max_new_tokens=100, output_len=0)

            drainer.update_draining_state(
                waiting_queue=[starving_a, starving_d],
                running_reqs=[running_b, running_c],
            )

            # B (smaller remaining tokens) should be drained for the most-starved adapter 'A'
            self.assertEqual(drainer.adapter_to_stats["B"].is_draining_for, "A")

            # C should be drained for the other starving adapter 'D'
            self.assertEqual(drainer.adapter_to_stats["C"].is_draining_for, "D")
```
**EN:** This test exercises `test_update_draining_marks_adapter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_draining_marks_adapter`。

### Lines 88-118: test case can schedule respects draining tolerance / 测试用例 can schedule respects draining tolerance
```python
    def test_can_schedule_respects_draining_tolerance(self):
        if is_in_ci():
            return

        with mock.patch("time.monotonic", return_value=MOCK_START_TIME):
            drainer = LoRADrainer(
                max_loras_per_batch=1, max_wait_time_secs=LORA_DRAIN_WAIT_THRESHOLD
            )

            wait_entry = MOCK_START_TIME - (LORA_DRAIN_WAIT_THRESHOLD + 0.01)
            starving_req = make_req("A", wait_entry, max_new_tokens=10)

            running_b = make_req("B", wait_entry, max_new_tokens=15, output_len=0)
            drainer.update_draining_state(
                waiting_queue=[starving_req],
                running_reqs=[running_b],
            )

            self.assertEqual(drainer.adapter_to_stats["B"].is_draining_for, "A")

            # max_new_tokens is less than running adapter B's max_new_tokens
            req_ok = make_req(
                lora_id="B", wait_queue_entry_time=0, max_new_tokens=10, output_len=0
            )
            self.assertTrue(drainer.can_schedule(req_ok))

            # max_new_tokens is more than running adapter B's max_new_tokens
            req_bad = make_req(
                lora_id="B", wait_queue_entry_time=0, max_new_tokens=20, output_len=0
            )
            self.assertFalse(drainer.can_schedule(req_bad))
```
**EN:** This test exercises `test_can_schedule_respects_draining_tolerance` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_can_schedule_respects_draining_tolerance`。

### Lines 120-127: test case batch splitting with drainer / 测试用例 batch splitting with drainer
```python
    def test_batch_splitting_with_drainer(self):
        run_lora_batch_splitting_equivalence_test(
            model_cases=CI_MULTI_LORA_MODELS,
            attention_backend="torch_native",
            disable_cuda_graph=True,
            disable_radix_cache=True,
            lora_drain_wait_threshold=LORA_DRAIN_WAIT_THRESHOLD,
        )
```
**EN:** This test exercises `test_batch_splitting_with_drainer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_splitting_with_drainer`。

### Lines 130-131: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `make_req`: This helper encapsulates `make_req` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `make_req`，以便周围测试复用准备、执行或校验逻辑。
- `TestLoRADrainer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRADrainer.test_update_draining_marks_adapter`: This test exercises `test_update_draining_marks_adapter` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_draining_marks_adapter`。
- `TestLoRADrainer.test_can_schedule_respects_draining_tolerance`: This test exercises `test_can_schedule_respects_draining_tolerance` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_can_schedule_respects_draining_tolerance`。
- `TestLoRADrainer.test_batch_splitting_with_drainer`: This test exercises `test_batch_splitting_with_drainer` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_splitting_with_drainer`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.lora.lora_drainer`, `sglang.srt.managers.schedule_batch`, `sglang.test.ci.ci_register`, `sglang.test.lora_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 131
