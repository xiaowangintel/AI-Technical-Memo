# test_qwen35.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_qwen35.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models qwen35 in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 8 gpu models qwen35 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

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

### Lines 12-12: Define module constants
```python
QWEN35_MODEL_PATH = "Qwen/Qwen3.5-397B-A17B-FP8"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 15-15: Define class TestQwen35
```python
class TestQwen35(unittest.TestCase):
```
**EN:** This declaration introduces the `TestQwen35` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen35` 测试类，并说明它通过继承承担的职责。

### Lines 16-23: Document the class `TestQwen35`
```python
    """Unified test class for Qwen3.5-397B-A17B performance and accuracy.

    Qwen3.5 is a 397B MoE VLM with 17B active params.
    Features hybrid reasoning, tool calling, and multimodal capabilities.
    Runs BOTH:
    - Performance test (using NightlyBenchmarkRunner)
    - Accuracy test (using run_eval with gsm8k)
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestQwen35`. It also runs an evaluation workflow to measure model quality.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestQwen35`的设计意图。 其中还会运行评测流程以衡量模型质量。

### Lines 25-76: Run test: qwen35
```python
    def test_qwen35(self):
        """Run performance and accuracy for Qwen3.5-397B-A17B."""
        base_args = [
            "--trust-remote-code",
            "--reasoning-parser=qwen3",
            "--tool-call-parser=qwen3_coder",
            "--mem-fraction-static=0.8",
        ]
        dp_args = ["--dp=8", "--enable-dp-attention"]
        mtp_args = [
            "--speculative-algorithm=EAGLE",
            "--speculative-num-steps=3",
            "--speculative-eagle-topk=1",
            "--speculative-num-draft-tokens=4",
            "--mamba-scheduler-strategy=extra_buffer",
        ]

        variants = [
            ModelLaunchSettings(
                QWEN35_MODEL_PATH,
                tp_size=8,
                extra_args=base_args,
                variant="TP8",
            ),
            ModelLaunchSettings(
                QWEN35_MODEL_PATH,
                tp_size=8,
                extra_args=base_args + dp_args,
                variant="TP8+DP8",
            ),
            ModelLaunchSettings(
                QWEN35_MODEL_PATH,
                tp_size=8,
                extra_args=base_args + dp_args + mtp_args,
                variant="TP8+DP8+MTP",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="Qwen3.5-397B-A17B",
            accuracy_params=AccuracyTestParams(
                dataset="gsm8k",
                baseline_accuracy=0.95,
                thinking_mode="qwen3",
                max_tokens=8192,
                num_examples=200,
            ),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_qwen35",
            ),
        )
```
**EN:** This test method exercises qwen35 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 qwen35 场景，并验证观测到的行为是否符合预期契约。

### Lines 79-80: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `run_eval`, `unittest.main`
