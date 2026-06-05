# test_nvidia_nemotron_3_super_nightly.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_nvidia_nemotron_3_super_nightly.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models nvidia nemotron 3 super nightly in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 8 gpu models nvidia nemotron 3 super nightly 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Import dependencies
```python
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.performance_test_runner import PerformanceTestParams
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import ModelLaunchSettings, is_blackwell_system
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 10-10: Register CI metadata
```python
register_cuda_ci(est_time=5400, suite="nightly-8-gpu-common", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 12-42: Define module constants
```python
NEMOTRON_3_SUPER_BF16_MODEL = "nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-BF16"
NEMOTRON_3_SUPER_NVFP4_MODEL = "nvidia/NVIDIA-Nemotron-3-Super-120B-A12B-NVFP4"

BASE_ARGS = [
    "--tp=8",
    "--trust-remote-code",
    "--reasoning-parser",
    "nemotron_3",
    "--tool-call-parser",
    "qwen3_coder",
    "--disable-radix-cache",
]

BF16_LOADER_ARGS = [
    "--model-loader-extra-config",
    '{"enable_multithread_load": true, "num_threads": 50}',
]

NVFP4_LOADER_ARGS = [
    "--model-loader-extra-config",
    '{"enable_multithread_load": true, "num_threads": 17}',
]

MTP_ARGS = [
    "--speculative-algorithm=EAGLE",
    "--speculative-num-steps=3",
    "--speculative-eagle-topk=1",
    "--speculative-num-draft-tokens=4",
    "--max-running-requests=512",
    "--mem-fraction-static=0.75",
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 45-45: Define module constants
```python
GSM8K_BASELINE = 0.935
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 48-48: Define class TestNvidiaNemotron3SuperNightly
```python
class TestNvidiaNemotron3SuperNightly(unittest.TestCase):
```
**EN:** This declaration introduces the `TestNvidiaNemotron3SuperNightly` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNvidiaNemotron3SuperNightly` 测试类，并说明它通过继承承担的职责。

### Lines 49-60: Document the class `TestNvidiaNemotron3SuperNightly`
```python
    """Unified nightly test class for Nemotron 3 Super 120B.

    BF16 variants (Hopper + Blackwell):
    - TP8, TP8+MTP

    NVFP4 variants (Blackwell only):
    - TP8, TP8+MTP

    Each variant runs BOTH:
    - Performance test (using NightlyBenchmarkRunner)
    - Accuracy test (using run_eval with gsm8k)
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNvidiaNemotron3SuperNightly`. It also runs an evaluation workflow to measure model quality.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNvidiaNemotron3SuperNightly`的设计意图。 其中还会运行评测流程以衡量模型质量。

### Lines 62-95: Run test: nemotron 3 super bf16
```python
    def test_nemotron_3_super_bf16(self):
        """Run performance and accuracy for all Nemotron 3 Super BF16 variants."""
        variants = [
            ModelLaunchSettings(
                NEMOTRON_3_SUPER_BF16_MODEL,
                tp_size=8,
                extra_args=BASE_ARGS + BF16_LOADER_ARGS,
                variant="TP8",
            ),
            ModelLaunchSettings(
                NEMOTRON_3_SUPER_BF16_MODEL,
                tp_size=8,
                extra_args=BASE_ARGS + BF16_LOADER_ARGS + MTP_ARGS,
                variant="TP8+MTP",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="Nemotron-3-Super-120B-BF16",
            accuracy_params=AccuracyTestParams(
                dataset="gsm8k",
                baseline_accuracy=GSM8K_BASELINE,
                num_examples=1314,
                num_threads=512,
                max_tokens=16000,
                temperature=1.0,
                top_p=0.95,
                repeat=1,
            ),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_nemotron_3_super_bf16",
            ),
        )
```
**EN:** This test method exercises nemotron 3 super bf16 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 nemotron 3 super bf16 场景，并验证观测到的行为是否符合预期契约。

### Lines 97-131: Run test: nemotron 3 super nvfp4
```python
    @unittest.skipIf(not is_blackwell_system(), "NVFP4 requires Blackwell")
    def test_nemotron_3_super_nvfp4(self):
        """Run performance and accuracy for all Nemotron 3 Super NVFP4 variants (Blackwell only)."""
        variants = [
            ModelLaunchSettings(
                NEMOTRON_3_SUPER_NVFP4_MODEL,
                tp_size=8,
                extra_args=BASE_ARGS + NVFP4_LOADER_ARGS,
                variant="TP8",
            ),
            ModelLaunchSettings(
                NEMOTRON_3_SUPER_NVFP4_MODEL,
                tp_size=8,
                extra_args=BASE_ARGS + NVFP4_LOADER_ARGS + MTP_ARGS,
                variant="TP8+MTP",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="Nemotron-3-Super-120B-NVFP4",
            accuracy_params=AccuracyTestParams(
                dataset="gsm8k",
                baseline_accuracy=GSM8K_BASELINE,
                num_examples=1314,
                num_threads=512,
                max_tokens=16000,
                temperature=1.0,
                top_p=0.95,
                repeat=1,
            ),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_nemotron_3_super_nvfp4",
            ),
        )
```
**EN:** This test method exercises nemotron 3 super nvfp4 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 nemotron 3 super nvfp4 场景，并验证观测到的行为是否符合预期契约。

### Lines 134-135: Expose unittest entrypoint
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
