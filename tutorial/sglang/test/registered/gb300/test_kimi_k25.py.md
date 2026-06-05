# test_kimi_k25.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/gb300/test_kimi_k25.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates kimi k25 behavior in SGLang's gb300 area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 gb300 领域中与 kimi k25 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

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

### Lines 9-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=7200, suite="nightly-4-gpu-gb300", nightly=True)

MODEL_PATH = "moonshotai/Kimi-K2.5"

COMMON_ARGS = [
    "--trust-remote-code",
    "--reasoning-parser=kimi_k2",
    "--tool-call-parser=kimi_k2",
    "--mem-fraction-static=0.8",
    "--enable-multimodal",
    "--enable-metrics",
]
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 23-23: class TestKimiK25 declaration / 类 TestKimiK25 声明
```python
class TestKimiK25(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 24-27: supporting statements / 辅助语句
```python
    """Kimi-K2.5 (native INT4) on GB300 (4x B200 NVL4, tp=4).

    No EAGLE/MTP support for Kimi-K2.5 — only TP and TP+DP+DPA variants.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 29-54: test case kimi k25 / 测试用例 kimi k25
```python
    def test_kimi_k25(self):
        variants = [
            ModelLaunchSettings(
                MODEL_PATH,
                tp_size=4,
                extra_args=COMMON_ARGS,
                variant="TP4",
            ),
            ModelLaunchSettings(
                MODEL_PATH,
                tp_size=4,
                extra_args=COMMON_ARGS + ["--dp-size=4", "--enable-dp-attention"],
                variant="TP4+DP4+DPA",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="Kimi-K2.5",
            accuracy_params=AccuracyTestParams(
                dataset="mmmu-pro", baseline_accuracy=0.69, repeat=1, max_tokens=32768
            ),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_gb300",
            ),
        )
```
**EN:** This test exercises `test_kimi_k25` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_k25`。

### Lines 57-58: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestKimiK25`: Kimi-K2.5 (native INT4) on GB300 (4x B200 NVL4, tp=4). / 用于组织相关测试、夹具或辅助方法。
- `TestKimiK25.test_kimi_k25`: This test exercises `test_kimi_k25` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_kimi_k25`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`

- **Total lines / 总行数**: 58
