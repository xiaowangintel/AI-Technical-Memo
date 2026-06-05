# test_vlm_perf_5090.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_vlm_perf_5090.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates vlm perf 5090 behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 vlm perf 5090 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
VLM Performance tests that work on 5090 (32GB) - VLM offline throughput and online latency tests.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-14: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    run_bench_serving,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 16-17: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=406, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=500, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 20-20: class TestVLMPerf5090 declaration / 类 TestVLMPerf5090 声明
```python
class TestVLMPerf5090(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 21-38: test case vlm offline throughput / 测试用例 vlm offline throughput
```python
    def test_vlm_offline_throughput(self):
        res = run_bench_serving(
            model=DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
            num_prompts=200,
            request_rate=float("inf"),
            other_server_args=[
                "--mem-fraction-static",
                "0.7",
            ],
            dataset_name="mmmu",
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_vlm_offline_throughput (5090)\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            self.assertGreater(res["output_throughput"], 2000)
```
**EN:** This test exercises `test_vlm_offline_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_offline_throughput`。

### Lines 40-59: test case vlm online latency / 测试用例 vlm online latency
```python
    def test_vlm_online_latency(self):
        res = run_bench_serving(
            model=DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
            num_prompts=250,
            request_rate=1,
            other_server_args=[
                "--mem-fraction-static",
                "0.7",
            ],
            dataset_name="mmmu",
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_vlm_online_latency (5090)\n"
                f"median_e2e_latency_ms: {res['median_e2e_latency_ms']:.2f} ms\n"
            )
            self.assertLess(res["median_e2e_latency_ms"], 16500)
            self.assertLess(res["median_ttft_ms"], 150)
            self.assertLess(res["median_itl_ms"], 8)
```
**EN:** This test exercises `test_vlm_online_latency` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_online_latency`。

### Lines 62-63: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestVLMPerf5090`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestVLMPerf5090.test_vlm_offline_throughput`: This test exercises `test_vlm_offline_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_offline_throughput`。
- `TestVLMPerf5090.test_vlm_online_latency`: This test exercises `test_vlm_online_latency` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_online_latency`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 63
