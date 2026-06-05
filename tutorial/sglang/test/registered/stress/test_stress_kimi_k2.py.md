# test_stress_kimi_k2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/stress/test_stress_kimi_k2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates stress kimi k2 behavior in SGLang's stress area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 stress 领域中与 stress kimi k2 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Stress test for Kimi-K2-Thinking model."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-8: module imports and dependencies / 模块导入与依赖
```python
import os
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.ci.ci_stress_utils import StressTestRunner
from sglang.test.test_utils import DEFAULT_URL_FOR_TEST
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `unittest`, `sglang.test.ci.ci_register`, `sglang.test.ci.ci_stress_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `unittest`, `sglang.test.ci.ci_register`, `sglang.test.ci.ci_stress_utils`。

### Lines 10-16: CI registration and metadata / CI 注册与元数据
```python
MODEL_PATH = "moonshotai/Kimi-K2-Thinking"
RANDOM_INPUT_LEN = 4096
RANDOM_OUTPUT_LEN = 512
OUTPUT_FILE = "stress_test_kimi_k2.jsonl"

# Register for CI - estimated 45 minutes
register_cuda_ci(est_time=2700, suite="stress")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 19-19: class TestStressKimiK2 declaration / 类 TestStressKimiK2 声明
```python
class TestStressKimiK2(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 20-32: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.num_prompts = int(os.environ.get("NUM_PROMPTS", "50000"))
        cls.duration_minutes = int(os.environ.get("DURATION_MINUTES", "45"))

        cls.runner = StressTestRunner(
            test_name="Kimi-K2-Thinking Stress Test",
            base_url=cls.base_url,
            num_prompts=cls.num_prompts,
            duration_minutes=cls.duration_minutes,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 34-55: test case stress kimi k2 / 测试用例 stress kimi k2
```python
    def test_stress_kimi_k2(self):
        try:
            success = self.runner.run_stress_test_for_model(
                model_path=self.model,
                random_input_len=RANDOM_INPUT_LEN,
                random_output_len=RANDOM_OUTPUT_LEN,
                output_file=OUTPUT_FILE,
                server_args=[
                    "--tp",
                    "8",
                    "--trust-remote-code",
                    "--tool-call-parser",
                    "kimi_k2",
                    "--reasoning-parser",
                    "kimi_k2",
                ],
            )

            self.assertTrue(success, f"Stress test failed for {self.model}")

        finally:
            self.runner.write_final_report()
```
**EN:** This test exercises `test_stress_kimi_k2` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stress_kimi_k2`。

### Lines 58-59: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStressKimiK2`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStressKimiK2.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestStressKimiK2.test_stress_kimi_k2`: This test exercises `test_stress_kimi_k2` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stress_kimi_k2`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.ci.ci_stress_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 59
