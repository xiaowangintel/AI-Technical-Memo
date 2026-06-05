# test_glm47_fp8_eval_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi35x/test_glm47_fp8_eval_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on glm47 fp8 eval mi35x in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 glm47 fp8 eval mi35x 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Document the module
```python
"""MI35x GLM-4.7-FP8 GSM8K Accuracy Evaluation Test (8-GPU)

Tests GLM-4.7-FP8 accuracy using GSM8K benchmark on MI35x.

Registry: nightly-amd-8-gpu-mi35x-glm47-fp8 suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 8-8: Import dependencies
```python
import os
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 11-12: Implement expr logic
```python
os.environ.setdefault("HF_HOME", "/data2/models/huggingface")
os.environ.setdefault("HF_HUB_CACHE", "/data2/models/huggingface/hub")
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 14-19: Import dependencies
```python
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import ModelLaunchSettings
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 22-26: Register CI metadata
```python
register_amd_ci(
    est_time=1800,
    suite="nightly-amd-8-gpu-mi35x-glm47-fp8",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 28-28: Define module constants
```python
GLM_4_7_FP8_MODEL_PATH = "zai-org/GLM-4.7-FP8"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 31-31: Define class TestGLM47FP8EvalMI35x
```python
class TestGLM47FP8EvalMI35x(unittest.TestCase):
```
**EN:** This declaration introduces the `TestGLM47FP8EvalMI35x` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGLM47FP8EvalMI35x` 测试类，并说明它通过继承承担的职责。

### Lines 32-32: Document the class `TestGLM47FP8EvalMI35x`
```python
    """GLM-4.7-FP8 GSM8K Accuracy Evaluation Test for MI35x."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGLM47FP8EvalMI35x`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGLM47FP8EvalMI35x`的设计意图。

### Lines 34-55: Run test: glm 47 fp8
```python
    def test_glm_47_fp8(self):
        """Run accuracy test for GLM-4.7-FP8."""
        base_args = [
            "--trust-remote-code",
            "--tool-call-parser=glm47",
            "--reasoning-parser=glm45",
        ]

        variants = [
            ModelLaunchSettings(
                GLM_4_7_FP8_MODEL_PATH,
                tp_size=8,
                extra_args=base_args,
                variant="TP8",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="GLM-4.7-FP8",
            accuracy_params=AccuracyTestParams(dataset="gsm8k", baseline_accuracy=0.92),
        )
```
**EN:** This test method exercises glm 47 fp8 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 glm 47 fp8 场景，并验证观测到的行为是否符合预期契约。

### Lines 58-59: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `unittest.main`
