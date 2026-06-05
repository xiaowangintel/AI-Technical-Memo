# test_glm_46_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_glm_46_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `glm 46 fp8` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual` 中的 `glm 46 fp8` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.performance_test_runner import PerformanceTestParams
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import ModelLaunchSettings

GLM_4_6_FP8_MODEL_PATH = "zai-org/GLM-4.6-FP8"
```
**EN:** This range imports `unittest`, `sglang.test.accuracy_test_runner`, `sglang.test.performance_test_runner` and `sglang.test.run_combined_tests`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. It delegates execution to `run_combined_tests`, which bundles launch settings with shared validation passes.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 它会调用 `run_combined_tests`，把启动配置与共享校验流程组合在一起。

### Lines 11-19: Class definition for TestGLM46FP8 / 类定义
```python
class TestGLM46FP8(unittest.TestCase):
    """Unified test class for GLM-4.6-FP8 performance and accuracy.

    Single variant with simple TP=8 configuration.
    Runs BOTH:
    - Performance test (using NightlyBenchmarkRunner)
    - Accuracy test (using run_eval with mgsm_en)
    """
```
**EN:** This range declares `TestGLM46FP8`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution. Representative call sites include `test`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 20-35: Test routines around test_glm_46_fp8_all_variants / 测试例程
```python
    def test_glm_46_fp8_all_variants(self):
        """Run performance and accuracy for GLM-4.6-FP8."""
        base_args = [
            "--tp=8",
            "--trust-remote-code",
        ]
        mtp_args = [
            "--speculative-algorithm=EAGLE",
            "--speculative-num-steps=3",
            "--speculative-eagle-topk=1",
            "--speculative-num-draft-tokens=4",
        ]

        variants = [
            ModelLaunchSettings(
                GLM_4_6_FP8_MODEL_PATH,
```
**EN:** This range defines concrete test routine(s) `test_glm_46_fp8_all_variants`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `ModelLaunchSettings`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 36-44: Scenario logic / 场景逻辑
```python
                tp_size=8,
                extra_args=base_args,
                variant="TP8",
            ),
            ModelLaunchSettings(
                GLM_4_6_FP8_MODEL_PATH,
                tp_size=8,
                extra_args=base_args + mtp_args,
                variant="TP8+MTP",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ModelLaunchSettings`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-55: Scenario logic / 场景逻辑
```python
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="GLM-4.6-FP8",
            accuracy_params=AccuracyTestParams(dataset="gsm8k", baseline_accuracy=0.80),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_glm_4_6_fp8",
            ),
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It delegates execution to `run_combined_tests`, which bundles launch settings with shared validation passes. The code also ties this block to evaluation or benchmark execution. Representative call sites include `run_combined_tests`, `AccuracyTestParams` and `PerformanceTestParams`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会调用 `run_combined_tests`，把启动配置与共享校验流程组合在一起。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 56-59: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.accuracy_test_runner`, `sglang.test.performance_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`
