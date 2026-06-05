# test_run_candidate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/auto_benchmark/test_run_candidate.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates run candidate behavior in SGLang's unit / auto benchmark area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / auto benchmark 领域中与 run candidate 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import sys
import time
import unittest
from pathlib import Path
from unittest import mock
```
**EN:** This block imports the modules needed by the rest of the file, including `sys`, `time`, `unittest`, `pathlib`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sys`, `time`, `unittest`, `pathlib`。

### Lines 7-10: module-level constants and configuration / 模块级常量与配置
```python
CURRENT_DIR = Path(__file__).resolve().parent
PARENT_DIR = CURRENT_DIR.parent
if str(PARENT_DIR) not in sys.path:
    sys.path.insert(0, str(PARENT_DIR))
```
**EN:** This block defines shared names such as `CURRENT_DIR`, `PARENT_DIR`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `CURRENT_DIR`, `PARENT_DIR` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 12-15: module imports and dependencies / 模块导入与依赖
```python
from auto_benchmark import AutoBenchmarkTestCase

from sglang.auto_benchmark_lib import SearchDeadlineExceeded, run_candidate
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `auto_benchmark`, `sglang.auto_benchmark_lib`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `auto_benchmark`, `sglang.auto_benchmark_lib`, `sglang.test.ci.ci_register`。

### Lines 17-18: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=6, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=6, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class TestAutoBenchmarkRunCandidate declaration / 类 TestAutoBenchmarkRunCandidate 声明
```python
class TestAutoBenchmarkRunCandidate(AutoBenchmarkTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `AutoBenchmarkTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `AutoBenchmarkTestCase`。

### Lines 22-36: test case run candidate binary search avoids rounding loop / 测试用例 run candidate binary search avoids rounding loop
```python
    def test_run_candidate_binary_search_avoids_rounding_loop(self):
        benchmark_cfg = {
            "qps": {"lower": 1.0, "upper": 1.00000001, "tolerance": 1e-12},
            "max_concurrency": [None],
        }
        calls = []

        with mock.patch(
            "sglang.auto_benchmark_lib.run_trial",
            side_effect=self._make_run_trial_side_effect(calls),
        ):
            records = run_candidate(**self._run_candidate_kwargs(benchmark_cfg))

        self.assertLess(len(calls), 40)
        self.assertEqual(len(records), len(calls))
```
**EN:** This test exercises `test_run_candidate_binary_search_avoids_rounding_loop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_binary_search_avoids_rounding_loop`。

### Lines 38-52: test case run candidate binary search respects max rounds / 测试用例 run candidate binary search respects max rounds
```python
    def test_run_candidate_binary_search_respects_max_rounds(self):
        benchmark_cfg = {
            "qps": {"lower": 1.0, "upper": 32.0, "tolerance": 1e-12, "max_rounds": 2},
            "max_concurrency": [None],
        }
        calls = []

        with mock.patch(
            "sglang.auto_benchmark_lib.run_trial",
            side_effect=self._make_run_trial_side_effect(calls),
        ):
            records = run_candidate(**self._run_candidate_kwargs(benchmark_cfg))

        self.assertEqual(len(calls), 2)
        self.assertEqual(len(records), 2)
```
**EN:** This test exercises `test_run_candidate_binary_search_respects_max_rounds` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_binary_search_respects_max_rounds`。

### Lines 54-67: test case run candidate stops when search budget is exhausted / 测试用例 run candidate stops when search budget is exhausted
```python
    def test_run_candidate_stops_when_search_budget_is_exhausted(self):
        benchmark_cfg = {
            "qps": {"lower": 1.0, "upper": 2.0, "tolerance": 0.1},
            "max_concurrency": [None],
        }

        with self.assertRaises(SearchDeadlineExceeded):
            run_candidate(
                **self._run_candidate_kwargs(
                    benchmark_cfg,
                    search_deadline=time.time() - 1.0,
                    search_budget_hours=0.1,
                )
            )
```
**EN:** This test exercises `test_run_candidate_stops_when_search_budget_is_exhausted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_stops_when_search_budget_is_exhausted`。

### Lines 69-94: test case run candidate resume skips existing fixed trials / 测试用例 run candidate resume skips existing fixed trials
```python
    def test_run_candidate_resume_skips_existing_fixed_trials(self):
        benchmark_cfg = {
            "qps": [1.0, 2.0],
            "max_concurrency": [None],
        }
        existing_records = [self._trial_record(1.0)]
        calls = []

        with mock.patch(
            "sglang.auto_benchmark_lib.run_trial",
            side_effect=self._make_run_trial_side_effect(
                calls,
                output_throughput=2.0,
                mean_ttft_ms=2.0,
                mean_tpot_ms=2.0,
            ),
        ):
            records = run_candidate(
                **self._run_candidate_kwargs(
                    benchmark_cfg,
                    existing_records=existing_records,
                )
            )

        self.assertEqual(calls, [2.0])
        self.assertEqual([record["requested_qps"] for record in records], [1.0, 2.0])
```
**EN:** This test exercises `test_run_candidate_resume_skips_existing_fixed_trials` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_resume_skips_existing_fixed_trials`。

### Lines 97-98: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestAutoBenchmarkRunCandidate`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAutoBenchmarkRunCandidate.test_run_candidate_binary_search_avoids_rounding_loop`: This test exercises `test_run_candidate_binary_search_avoids_rounding_loop` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_binary_search_avoids_rounding_loop`。
- `TestAutoBenchmarkRunCandidate.test_run_candidate_binary_search_respects_max_rounds`: This test exercises `test_run_candidate_binary_search_respects_max_rounds` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_binary_search_respects_max_rounds`。
- `TestAutoBenchmarkRunCandidate.test_run_candidate_stops_when_search_budget_is_exhausted`: This test exercises `test_run_candidate_stops_when_search_budget_is_exhausted` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_stops_when_search_budget_is_exhausted`。
- `TestAutoBenchmarkRunCandidate.test_run_candidate_resume_skips_existing_fixed_trials`: This test exercises `test_run_candidate_resume_skips_existing_fixed_trials` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_run_candidate_resume_skips_existing_fixed_trials`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `sys`, `time`, `unittest`, `pathlib`
- **Third-party modules / 第三方模块**: `auto_benchmark`
- **Internal modules / 内部模块**: `sglang.auto_benchmark_lib`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 98
