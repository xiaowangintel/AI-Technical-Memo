# test_bench_serving_2gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_bench_serving_2gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates bench serving 2gpu behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 bench serving 2gpu 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
Performance tests for 2-GPU that need large GPUs (H200 80GB) - MoE and Pipeline Parallel tests.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-15: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_MOE_MODEL_NAME_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    run_bench_serving,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 17-18: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=721, stage="extra-a", runner_config="2-gpu-large")
register_amd_ci(est_time=1450, suite="stage-b-test-2-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 21-21: class TestBenchServing2GPU declaration / 类 TestBenchServing2GPU 声明
```python
class TestBenchServing2GPU(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 22-38: test case moe offline throughput default / 测试用例 moe offline throughput default
```python
    def test_moe_offline_throughput_default(self):
        res = run_bench_serving(
            model=DEFAULT_MOE_MODEL_NAME_FOR_TEST,
            num_prompts=300,
            request_rate=float("inf"),
            other_server_args=["--tp", "2"],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_moe_offline_throughput_default\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 2100)
            else:
                self.assertGreater(res["output_throughput"], 2200)
```
**EN:** This test exercises `test_moe_offline_throughput_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_offline_throughput_default`。

### Lines 40-56: test case moe offline throughput without radix cache / 测试用例 moe offline throughput without radix cache
```python
    def test_moe_offline_throughput_without_radix_cache(self):
        res = run_bench_serving(
            model=DEFAULT_MOE_MODEL_NAME_FOR_TEST,
            num_prompts=300,
            request_rate=float("inf"),
            other_server_args=["--tp", "2", "--disable-radix-cache"],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_moe_offline_throughput_without_radix_cache\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 2100)
            else:
                self.assertGreater(res["output_throughput"], 2200)
```
**EN:** This test exercises `test_moe_offline_throughput_without_radix_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_offline_throughput_without_radix_cache`。

### Lines 58-75: test case pp offline throughput default decode / 测试用例 pp offline throughput default decode
```python
    def test_pp_offline_throughput_default_decode(self):
        res = run_bench_serving(
            model=DEFAULT_MOE_MODEL_NAME_FOR_TEST,
            num_prompts=1000,
            request_rate=float("inf"),
            random_input_len=1,
            random_output_len=1024,
            other_server_args=["--pp-size", "2"],
            need_warmup=True,
            seed=42,
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_pp_offline_throughput_default_decode\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            self.assertGreater(res["output_throughput"], 6700)
```
**EN:** This test exercises `test_pp_offline_throughput_default_decode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pp_offline_throughput_default_decode`。

### Lines 77-104: test case pp long context prefill / 测试用例 pp long context prefill
```python
    def test_pp_long_context_prefill(self):
        res = run_bench_serving(
            model="meta-llama/Llama-3.3-70B-Instruct",
            num_prompts=4,
            request_rate=float("inf"),
            random_input_len=128000,
            random_output_len=1,
            dataset_name="random",
            other_server_args=[
                "--quantization",
                "fp8",
                "--pp-size",
                "2",
            ]
            + (["--mem-fraction-static", "0.7"] if is_in_amd_ci() else []),
            need_warmup=False,
            seed=42,
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_pp_long_context_latency_prefill\n"
                f"input_throughput: {res['input_throughput']:.2f} ms\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["input_throughput"], 3000)
            else:
                self.assertGreater(res["input_throughput"], 4000)
```
**EN:** This test exercises `test_pp_long_context_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pp_long_context_prefill`。

### Lines 107-108: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBenchServing2GPU`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBenchServing2GPU.test_moe_offline_throughput_default`: This test exercises `test_moe_offline_throughput_default` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_offline_throughput_default`。
- `TestBenchServing2GPU.test_moe_offline_throughput_without_radix_cache`: This test exercises `test_moe_offline_throughput_without_radix_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_offline_throughput_without_radix_cache`。
- `TestBenchServing2GPU.test_pp_offline_throughput_default_decode`: This test exercises `test_pp_offline_throughput_default_decode` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pp_offline_throughput_default_decode`。
- `TestBenchServing2GPU.test_pp_long_context_prefill`: This test exercises `test_pp_long_context_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pp_long_context_prefill`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 108
