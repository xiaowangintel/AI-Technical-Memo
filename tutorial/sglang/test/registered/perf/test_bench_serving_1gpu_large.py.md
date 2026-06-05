# test_bench_serving_1gpu_large.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_bench_serving_1gpu_large.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates bench serving 1gpu large behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 bench serving 1gpu large 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
Performance tests for single GPU that need H200 (80GB) - FP8 and EAGLE tests.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-18: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.utils import is_hip
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE,
    DEFAULT_MODEL_NAME_FOR_TEST_FP8,
    DEFAULT_TARGET_MODEL_EAGLE,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    run_bench_serving,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 20-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=286, stage="extra-a", runner_config="1-gpu-large")
register_amd_ci(est_time=300, suite="stage-b-test-1-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestBenchServing1GPULarge declaration / 类 TestBenchServing1GPULarge 声明
```python
class TestBenchServing1GPULarge(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-41: test case offline throughput default fp8 / 测试用例 offline throughput default fp8
```python
    def test_offline_throughput_default_fp8(self):
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST_FP8,
            num_prompts=500,
            request_rate=float("inf"),
            other_server_args=[],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_offline_throughput_default_fp8\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 3500)
            else:
                self.assertGreater(res["output_throughput"], 4300)
```
**EN:** This test exercises `test_offline_throughput_default_fp8` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_default_fp8`。

### Lines 43-80: test case online latency eagle / 测试用例 online latency eagle
```python
    @unittest.skipIf(is_hip(), "Skip Eagle test for ROCm")
    def test_online_latency_eagle(self):
        res = run_bench_serving(
            model=DEFAULT_TARGET_MODEL_EAGLE,
            num_prompts=300,
            request_rate=8,
            sharegpt_context_len=3072,
            disable_ignore_eos=True,
            dataset_name="sharegpt",
            other_server_args=[
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-draft-model-path",
                DEFAULT_DRAFT_MODEL_EAGLE,
                "--speculative-num-steps",
                "5",
                "--speculative-eagle-topk",
                "4",
                "--speculative-num-draft-tokens",
                "16",
                "--mem-fraction-static",
                "0.7",
            ],
            need_warmup=True,
            seed=42,
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_online_latency_eagle\n"
                f"median_e2e_latency_ms: {res['median_e2e_latency_ms']:.2f} ms\n"
                f"accept_length: {res['accept_length']:.2f} \n"
            )
            if is_in_amd_ci():
                self.assertLess(res["median_e2e_latency_ms"], 1800)
            else:
                self.assertLess(res["median_e2e_latency_ms"], 900)
            self.assertGreater(res["accept_length"], 3.0)
```
**EN:** This test exercises `test_online_latency_eagle` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_latency_eagle`。

### Lines 83-84: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBenchServing1GPULarge`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBenchServing1GPULarge.test_offline_throughput_default_fp8`: This test exercises `test_offline_throughput_default_fp8` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_default_fp8`。
- `TestBenchServing1GPULarge.test_online_latency_eagle`: This test exercises `test_online_latency_eagle` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_latency_eagle`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 84
