# test_glm_51_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_glm_51_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models glm 51 fp8 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 8 gpu models glm 51 fp8 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Import dependencies
```python
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.performance_test_runner import PerformanceTestParams
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import ModelLaunchSettings
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 10-10: Register CI metadata
```python
register_cuda_ci(est_time=1800, suite="nightly-8-gpu-common", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 12-27: Define module constants
```python
GLM_51_FP8_MODEL_PATH = "zai-org/GLM-5.1-FP8"

COMMON_ARGS = [
    "--trust-remote-code",
    "--reasoning-parser=glm45",
    "--tool-call-parser=glm47",
    "--mem-fraction-static=0.85",
    "--enable-metrics",
]

MTP_ARGS = [
    "--speculative-algorithm=EAGLE",
    "--speculative-num-steps=3",
    "--speculative-eagle-topk=1",
    "--speculative-num-draft-tokens=4",
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 30-30: Define class TestGlm51Fp8
```python
class TestGlm51Fp8(unittest.TestCase):
```
**EN:** This declaration introduces the `TestGlm51Fp8` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGlm51Fp8` 测试类，并说明它通过继承承担的职责。

### Lines 31-31: Document the class `TestGlm51Fp8`
```python
    """GLM-5.1 FP8 on H200/B200 (8-GPU, tp=8)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGlm51Fp8`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGlm51Fp8`的设计意图。

### Lines 33-65: Run test: glm51 fp8
```python
    def test_glm51_fp8(self):
        dp_args = ["--dp=8", "--enable-dp-attention"]

        variants = [
            ModelLaunchSettings(
                GLM_51_FP8_MODEL_PATH,
                tp_size=8,
                extra_args=COMMON_ARGS,
                variant="TP8",
            ),
            ModelLaunchSettings(
                GLM_51_FP8_MODEL_PATH,
                tp_size=8,
                extra_args=COMMON_ARGS + dp_args,
                variant="TP8+DP8",
            ),
            ModelLaunchSettings(
                GLM_51_FP8_MODEL_PATH,
                tp_size=8,
                extra_args=COMMON_ARGS + dp_args + MTP_ARGS,
                variant="TP8+DP8+MTP",
                env={"SGLANG_ENABLE_SPEC_V2": "1"},
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="GLM-5.1-FP8",
            accuracy_params=AccuracyTestParams(dataset="gsm8k", baseline_accuracy=0.92),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_glm_51_fp8",
            ),
        )
```
**EN:** This test method exercises glm51 fp8 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 glm51 fp8 场景，并验证观测到的行为是否符合预期契约。

### Lines 68-69: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
