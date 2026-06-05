# test_deepseek_v32_perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/nightly/test_deepseek_v32_perf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `deepseek v32 perf` scenario in `test/manual/nightly`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/nightly` 中的 `deepseek v32 perf` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

from sglang.test.nightly_utils import NightlyBenchmarkRunner
from sglang.test.test_utils import DEFAULT_URL_FOR_TEST, _parse_int_list_env

DEEPSEEK_V32_MODEL_PATH = "deepseek-ai/DeepSeek-V3.2"
PROFILE_DIR = "performance_profiles_deepseek_v32"
```
**EN:** This range imports `unittest`, `sglang.test.nightly_utils` and `sglang.test.test_utils`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 10-11: Class definition for TestNightlyDeepseekV32Performance / 类定义
```python
class TestNightlyDeepseekV32Performance(unittest.TestCase):
    @classmethod
```
**EN:** This range declares `TestNightlyDeepseekV32Performance`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 12-34: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DEEPSEEK_V32_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.batch_sizes = [1, 1, 8, 16, 64]
        cls.input_lens = tuple(_parse_int_list_env("NIGHTLY_INPUT_LENS", "4096"))
        cls.output_lens = tuple(_parse_int_list_env("NIGHTLY_OUTPUT_LENS", "512"))

        # Define variant configurations
        cls.variants = [
            {
                "name": "basic",
                "other_args": [
                    "--trust-remote-code",
                    "--tp",
                    "8",
                    "--dp",
                    "8",
                    "--enable-dp-attention",
                    "--model-loader-extra-config",
                    '{"enable_multithread_load": true}',
                ],
            },
            {
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `_parse_int_list_env`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 35-36: Scenario logic / 场景逻辑
```python
                "name": "mtp",
                "other_args": [
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 37-59: Scenario logic / 场景逻辑
```python
                    "--trust-remote-code",
                    "--tp",
                    "8",
                    "--dp",
                    "8",
                    "--enable-dp-attention",
                    "--speculative-algorithm",
                    "EAGLE",
                    "--speculative-num-steps",
                    "3",
                    "--speculative-eagle-topk",
                    "1",
                    "--speculative-num-draft-tokens",
                    "4",
                    "--mem-frac",
                    "0.7",
                    "--model-loader-extra-config",
                    '{"enable_multithread_load": true}',
                ],
            },
            {
                "name": "nsa",
                "other_args": [
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 60-61: Scenario logic / 场景逻辑
```python
                    "--trust-remote-code",
                    "--tp",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 62-84: Scenario logic / 场景逻辑
```python
                    "8",
                    "--dp",
                    "8",
                    "--enable-dp-attention",
                    "--attention-backend",
                    "nsa",
                    "--nsa-prefill-backend",
                    "flashmla_sparse",
                    "--nsa-decode-backend",
                    "flashmla_kv",
                    "--model-loader-extra-config",
                    '{"enable_multithread_load": true}',
                ],
            },
            {
                "name": "pure_tp",
                "other_args": [
                    "--trust-remote-code",
                    "--tp",
                    "8",
                    "--attention-backend",
                    "nsa",
                    "--nsa-prefill-backend",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 85-86: Scenario logic / 场景逻辑
```python
                    "flashmla_sparse",
                    "--nsa-decode-backend",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 87-95: Request and response handling / 请求与响应处理
```python
                    "flashmla_kv",
                    "--model-loader-extra-config",
                    '{"enable_multithread_load": true}',
                ],
            },
        ]

        cls.runner = NightlyBenchmarkRunner(PROFILE_DIR, cls.__name__, cls.base_url)
        cls.runner.setup_profile_directory()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Representative call sites include `NightlyBenchmarkRunner` and `setup_profile_directory`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 97-109: Test routines around test_bench_one_batch / 测试例程
```python
    def test_bench_one_batch(self):
        failed_variants = []

        try:
            for variant_config in self.variants:
                with self.subTest(variant=variant_config["name"]):
                    results, success = self.runner.run_benchmark_for_model(
                        model_path=self.model,
                        batch_sizes=self.batch_sizes,
                        input_lens=self.input_lens,
                        output_lens=self.output_lens,
                        other_args=variant_config["other_args"],
                        variant=variant_config["name"],
```
**EN:** This range defines concrete test routine(s) `test_bench_one_batch`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. Representative call sites include `subTest` and `run_benchmark_for_model`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 110-121: Assertions and result checks / 断言与结果检查
```python
                    )

                    if not success:
                        failed_variants.append(variant_config["name"])

                    self.runner.add_report(results, variant=variant_config["name"])
        finally:
            self.runner.write_final_report()

        if failed_variants:
            raise AssertionError(
                f"Benchmark failed for {self.model} with the following variants: "
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Assertions in this block enforce the intended outcome. Representative call sites include `append`, `add_report`, `write_final_report` and `AssertionError`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 122-123: Scenario logic / 场景逻辑
```python
                f"{', '.join(failed_variants)}"
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 124-127: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.nightly_utils`, `sglang.test.test_utils`
