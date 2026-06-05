# test_dp_budget.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_dp_budget.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates dp budget behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 dp budget 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: supporting statements / 辅助语句
```python
"""Unit tests for DPBudget — field mapping regression guard.

This PR changed DPBudget.update_budget to read num_running_reqs +
num_waiting_reqs and num_total_tokens from the new GetLoadsReqOutput.
These tests lock in that mapping. Pre-existing dispatch logic is not
retested here — it's covered by DP balance integration tests.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 9-13: module imports and dependencies / 模块导入与依赖
```python
import dataclasses
import unittest

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase, maybe_stub_sgl_kernel
```
**EN:** This block imports the modules needed by the rest of the file, including `dataclasses`, `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `dataclasses`, `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 15-15: supporting statements / 辅助语句
```python
maybe_stub_sgl_kernel()
```
**EN:** This block performs supporting work through calls such as `maybe_stub_sgl_kernel`, preparing state for nearby definitions.
**CN:** 该代码块通过 `maybe_stub_sgl_kernel` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 17-18: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.managers.data_parallel_controller import DPBudget
from sglang.srt.managers.io_struct import GetLoadsReqOutput, WatchLoadUpdateReq
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.managers.data_parallel_controller`, `sglang.srt.managers.io_struct`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.managers.data_parallel_controller`, `sglang.srt.managers.io_struct`。

### Lines 20-36: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=11, suite="base-a-test-cpu")


_BASE_LOAD = GetLoadsReqOutput(
    dp_rank=0,
    timestamp=0.0,
    num_running_reqs=0,
    num_waiting_reqs=0,
    num_used_tokens=0,
    num_total_tokens=0,
    max_total_num_tokens=4096,
    token_usage=0.0,
    gen_throughput=0.0,
    cache_hit_rate=0.0,
    utilization=0.0,
    max_running_requests=128,
)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci, GetLoadsReqOutput.
**CN:** 该代码块通过 register_cpu_ci, GetLoadsReqOutput 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 39-40: function load / 函数 load
```python
def _load(**overrides) -> GetLoadsReqOutput:
    return dataclasses.replace(_BASE_LOAD, **overrides)
```
**EN:** This block implements `_load` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_load`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-43: class TestDPBudgetUpdateBudget declaration / 类 TestDPBudgetUpdateBudget 声明
```python
class TestDPBudgetUpdateBudget(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 44-54: test case maps running plus waiting to total requests / 测试用例 maps running plus waiting to total requests
```python
    def test_maps_running_plus_waiting_to_total_requests(self):
        budget = DPBudget(dp_size=2)
        budget.update_budget(
            WatchLoadUpdateReq(
                loads=[
                    _load(dp_rank=0, num_running_reqs=3, num_waiting_reqs=2),
                    _load(dp_rank=1, num_running_reqs=5, num_waiting_reqs=1),
                ]
            )
        )
        self.assertEqual(budget.total_requests, [5, 6])
```
**EN:** This test exercises `test_maps_running_plus_waiting_to_total_requests` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_maps_running_plus_waiting_to_total_requests`。

### Lines 56-68: test case maps num total tokens not num used tokens / 测试用例 maps num total tokens not num used tokens
```python
    def test_maps_num_total_tokens_not_num_used_tokens(self):
        # Reads num_total_tokens (used + pending prefill), NOT num_used_tokens.
        # A silent swap here would break DP balance for long-prompt workloads.
        budget = DPBudget(dp_size=2)
        budget.update_budget(
            WatchLoadUpdateReq(
                loads=[
                    _load(dp_rank=0, num_used_tokens=100, num_total_tokens=150),
                    _load(dp_rank=1, num_used_tokens=80, num_total_tokens=80),
                ]
            )
        )
        self.assertEqual(budget.total_tokens, [150, 80])
```
**EN:** This test exercises `test_maps_num_total_tokens_not_num_used_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_maps_num_total_tokens_not_num_used_tokens`。

### Lines 70-87: test case partial update only affects reported rank / 测试用例 partial update only affects reported rank
```python
    def test_partial_update_only_affects_reported_rank(self):
        budget = DPBudget(dp_size=3)
        budget.total_requests = [10, 20, 30]
        budget.total_tokens = [100, 200, 300]
        budget.update_budget(
            WatchLoadUpdateReq(
                loads=[
                    _load(
                        dp_rank=1,
                        num_running_reqs=1,
                        num_waiting_reqs=1,
                        num_total_tokens=50,
                    )
                ]
            )
        )
        self.assertEqual(budget.total_requests, [10, 2, 30])
        self.assertEqual(budget.total_tokens, [100, 50, 300])
```
**EN:** This test exercises `test_partial_update_only_affects_reported_rank` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_update_only_affects_reported_rank`。

### Lines 90-91: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_load`: This block implements `_load` and captures one focused piece of the module's behavior. / 该代码块实现 `_load`，承担模块行为中的一个聚焦逻辑片段。
- `TestDPBudgetUpdateBudget`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDPBudgetUpdateBudget.test_maps_running_plus_waiting_to_total_requests`: This test exercises `test_maps_running_plus_waiting_to_total_requests` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_maps_running_plus_waiting_to_total_requests`。
- `TestDPBudgetUpdateBudget.test_maps_num_total_tokens_not_num_used_tokens`: This test exercises `test_maps_num_total_tokens_not_num_used_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_maps_num_total_tokens_not_num_used_tokens`。
- `TestDPBudgetUpdateBudget.test_partial_update_only_affects_reported_rank`: This test exercises `test_partial_update_only_affects_reported_rank` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_partial_update_only_affects_reported_rank`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.srt.managers.data_parallel_controller`, `sglang.srt.managers.io_struct`

- **Total lines / 总行数**: 91
