# test_bench_one_batch_2gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_bench_one_batch_2gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates bench one batch 2gpu behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 bench one batch 2gpu 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_MOE_MODEL_NAME_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    run_bench_offline_throughput,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 14-15: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=209, stage="extra-a", runner_config="2-gpu-large")
register_amd_ci(est_time=630, suite="stage-b-test-2-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 18-19: class TestBenchOneBatch2GPU declaration / 类 TestBenchOneBatch2GPU 声明
```python
class TestBenchOneBatch2GPU(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 20-33: test case moe tp2 bs1 / 测试用例 moe tp2 bs1
```python
    def test_moe_tp2_bs1(self):
        output_throughput = run_bench_offline_throughput(
            DEFAULT_MOE_MODEL_NAME_FOR_TEST, ["--tp", "2", "--cuda-graph-max-bs", "2"]
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_moe_tp2_bs1 (Mixtral-8x7B)\n"
                f"output_throughput: {output_throughput:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(output_throughput, 85)
            else:
                self.assertGreater(output_throughput, 125)
```
**EN:** This test exercises `test_moe_tp2_bs1` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_tp2_bs1`。

### Lines 35-49: test case torch compile tp2 bs1 / 测试用例 torch compile tp2 bs1
```python
    def test_torch_compile_tp2_bs1(self):
        output_throughput = run_bench_offline_throughput(
            DEFAULT_MODEL_NAME_FOR_TEST,
            ["--tp", "2", "--enable-torch-compile", "--cuda-graph-max-bs", "2"],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_torch_compile_tp2_bs1 (Mixtral-8x7B)\n"
                f"output_throughput: {output_throughput:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(output_throughput, 200)
            else:
                self.assertGreater(output_throughput, 220)
```
**EN:** This test exercises `test_torch_compile_tp2_bs1` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_torch_compile_tp2_bs1`。

### Lines 52-53: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBenchOneBatch2GPU`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBenchOneBatch2GPU.test_moe_tp2_bs1`: This test exercises `test_moe_tp2_bs1` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_moe_tp2_bs1`。
- `TestBenchOneBatch2GPU.test_torch_compile_tp2_bs1`: This test exercises `test_torch_compile_tp2_bs1` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_torch_compile_tp2_bs1`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 53
