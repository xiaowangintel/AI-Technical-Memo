# test_text_models_perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_text_models_perf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates text models perf behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 text models perf 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.nightly_utils import NightlyBenchmarkRunner
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    ModelLaunchSettings,
    _parse_int_list_env,
    parse_models,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.nightly_utils`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.nightly_utils`, `sglang.test.test_utils`。

### Lines 12-14: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=3600, suite="nightly-perf-text-2-gpu", nightly=True)

PROFILE_DIR = "performance_profiles_text_models"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-17: class TestNightlyTextModelsPerformance declaration / 类 TestNightlyTextModelsPerformance 声明
```python
class TestNightlyTextModelsPerformance(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 18-35: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.models = []
        # TODO: replace with DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1 or other model lists
        for model_path in parse_models("meta-llama/Llama-3.1-8B-Instruct"):
            cls.models.append(ModelLaunchSettings(model_path, tp_size=1))
        for model_path in parse_models("Qwen/Qwen2-57B-A14B-Instruct"):
            cls.models.append(ModelLaunchSettings(model_path, tp_size=2))
        # (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP1), False, False),
        # (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_TP2), False, True),
        # (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP1), True, False),
        # (parse_models(DEFAULT_MODEL_NAME_FOR_NIGHTLY_EVAL_FP8_TP2), True, True),
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.batch_sizes = [1, 1, 8, 16, 64]
        cls.input_lens = tuple(_parse_int_list_env("NIGHTLY_INPUT_LENS", "4096"))
        cls.output_lens = tuple(_parse_int_list_env("NIGHTLY_OUTPUT_LENS", "512"))
        cls.runner = NightlyBenchmarkRunner(PROFILE_DIR, cls.__name__, cls.base_url)
        cls.runner.setup_profile_directory()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 37-58: test case bench one batch / 测试用例 bench one batch
```python
    def test_bench_one_batch(self):
        all_model_succeed = True

        for model_setup in self.models:
            with self.subTest(model=model_setup.model_path):
                results, success, _ = self.runner.run_benchmark_for_model(
                    model_path=model_setup.model_path,
                    batch_sizes=self.batch_sizes,
                    input_lens=self.input_lens,
                    output_lens=self.output_lens,
                    other_args=model_setup.extra_args,
                )

                if not success:
                    all_model_succeed = False

                self.runner.add_report(results)

        self.runner.write_final_report()

        if not all_model_succeed:
            raise AssertionError("Some models failed the perf tests.")
```
**EN:** This test exercises `test_bench_one_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bench_one_batch`。

### Lines 61-62: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestNightlyTextModelsPerformance`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNightlyTextModelsPerformance.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestNightlyTextModelsPerformance.test_bench_one_batch`: This test exercises `test_bench_one_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bench_one_batch`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.nightly_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 62
