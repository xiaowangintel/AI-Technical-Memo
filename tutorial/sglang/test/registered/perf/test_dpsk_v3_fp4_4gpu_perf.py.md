# test_dpsk_v3_fp4_4gpu_perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_dpsk_v3_fp4_4gpu_perf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates dpsk v3 fp4 4gpu perf behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 dpsk v3 fp4 4gpu perf 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.performance_test_runner import PerformanceTestParams
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import ModelLaunchSettings
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`。

### Lines 8-9: supporting source context / 辅助源码上下文
```python

# Runs on B200 via nightly-4-gpu-b200 suite
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 10-12: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=2000, suite="nightly-4-gpu-b200", nightly=True)

FULL_DEEPSEEK_V3_FP4_MODEL_PATH = "nvidia/DeepSeek-V3-0324-FP4"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-15: class TestDeepseekR1FP4Unified declaration / 类 TestDeepseekR1FP4Unified 声明
```python
class TestDeepseekR1FP4Unified(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 16-25: supporting statements / 辅助语句
```python
    """Unified test class for DeepSeek-V3-0324-FP4 performance and accuracy.

    Two variants:
    - basic: Standard TP=4
    - mtp: TP=4 + EAGLE speculative decoding

    Each variant runs BOTH:
    - Performance test (using NightlyBenchmarkRunner)
    - Accuracy test (using run_eval with mgsm_en)
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 27-74: test case deepseek r1 fp4 all variants / 测试用例 deepseek r1 fp4 all variants
```python
    def test_deepseek_r1_fp4_all_variants(self):
        """Run performance and accuracy for all DeepSeek-R1-0528-NVFP4-v2 variants."""
        # Define base arguments shared by most variants
        base_args = [
            "--tp=4",
            "--trust-remote-code",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
        ]
        mtp_args = [
            "--speculative-algorithm=EAGLE",
            "--speculative-num-steps=3",
            "--speculative-eagle-topk=1",
            "--speculative-num-draft-tokens=4",
            "--mem-frac=0.7",
        ]

        variants = [
            # Variant: "basic" - Standard TP=4
            ModelLaunchSettings(
                FULL_DEEPSEEK_V3_FP4_MODEL_PATH,
                tp_size=4,
                extra_args=base_args,
                variant="TP4",
            ),
            # Variant: "mtp" - TP=4 + EAGLE speculative decoding
            ModelLaunchSettings(
                FULL_DEEPSEEK_V3_FP4_MODEL_PATH,
                tp_size=4,
                extra_args=base_args + mtp_args,
                variant="TP4+MTP",
                env={"SGLANG_ENABLE_SPEC_V2": "1"},
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="DeepSeek-V3-0324-FP4 Unified",
            accuracy_params=AccuracyTestParams(
                dataset="gsm8k",
                baseline_accuracy=0.935,
                num_examples=200,
                api="completion",
            ),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_deepseek_v3_fp4",
            ),
        )
```
**EN:** Run performance and accuracy for all DeepSeek-R1-0528-NVFP4-v2 variants. This test exercises `test_deepseek_r1_fp4_all_variants` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Run performance and accuracy for all DeepSeek-R1-0528-NVFP4-v2 variants. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_r1_fp4_all_variants`。

### Lines 77-78: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDeepseekR1FP4Unified`: Unified test class for DeepSeek-V3-0324-FP4 performance and accuracy. / 用于组织相关测试、夹具或辅助方法。
- `TestDeepseekR1FP4Unified.test_deepseek_r1_fp4_all_variants`: Run performance and accuracy for all DeepSeek-R1-0528-NVFP4-v2 variants. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_r1_fp4_all_variants`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`

- **Total lines / 总行数**: 78
