# test_mistral_large3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_mistral_large3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models mistral large3 in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 8 gpu models mistral large3 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies
```python
import os
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.performance_test_runner import PerformanceTestParams
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import ModelLaunchSettings, is_blackwell_system
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 12-12: Register CI metadata
```python
register_cuda_ci(est_time=3000, suite="nightly-8-gpu-common", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 14-16: Define module constants
```python
MISTRAL_LARGE3_FP8_MODEL_PATH = "mistralai/Mistral-Large-3-675B-Instruct-2512"
MISTRAL_LARGE3_NVFP4_MODEL_PATH = "mistralai/Mistral-Large-3-675B-Instruct-2512-NVFP4"
MISTRAL_LARGE3_EAGLE_MODEL_PATH = "mistralai/Mistral-Large-3-675B-Instruct-2512-Eagle"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 19-20: Define class TestMistralLarge3
```python
@unittest.skipIf(not is_blackwell_system(), "Requires B200")
class TestMistralLarge3(unittest.TestCase):
```
**EN:** This declaration introduces the `TestMistralLarge3` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMistralLarge3` 测试类，并说明它通过继承承担的职责。

### Lines 21-31: Document the class `TestMistralLarge3`
```python
    """Unified test class for Mistral-Large-3 performance and accuracy.

    Three variants:
    - basic: FP8 model + TP=8 + trtllm_mla backend
    - eagle: basic + EAGLE speculative decoding with draft model
    - nvfp4: NVFP4 model + TP=8 + trtllm_mla backend

    Each variant runs BOTH:
    - Performance test (using NightlyBenchmarkRunner)
    - Accuracy test (using run_eval with mgsm_en)
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMistralLarge3`. It also runs an evaluation workflow to measure model quality.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMistralLarge3`的设计意图。 其中还会运行评测流程以衡量模型质量。

### Lines 33-36: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        # Set environment variable to disable JIT DeepGemm
        os.environ["SGLANG_ENABLE_JIT_DEEPGEMM"] = "0"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 38-42: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        # Clean up environment variable
        if "SGLANG_ENABLE_JIT_DEEPGEMM" in os.environ:
            del os.environ["SGLANG_ENABLE_JIT_DEEPGEMM"]
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 44-94: Run test: mistral large3 all variants
```python
    def test_mistral_large3_all_variants(self):
        """Run performance and accuracy for all Mistral-Large-3 variants."""
        base_args = [
            "--tp=8",
            "--attention-backend=trtllm_mla",
            "--moe-runner-backend=flashinfer_trtllm",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
            "--chat-template=mistral",
        ]
        eagle_args = [
            "--speculative-algorithm=EAGLE",
            f"--speculative-draft-model-path={MISTRAL_LARGE3_EAGLE_MODEL_PATH}",
            "--speculative-num-steps=3",
            "--speculative-eagle-topk=1",
            "--speculative-num-draft-tokens=4",
            "--kv-cache-dtype=auto",
        ]

        variants = [
            # Variant: "basic" - FP8 model + TP=8 + trtllm_mla backend
            ModelLaunchSettings(
                MISTRAL_LARGE3_FP8_MODEL_PATH,
                tp_size=8,
                extra_args=base_args,
                variant="TP8",
            ),
            # Variant: "eagle" - FP8 model + TP=8 + trtllm_mla + EAGLE with draft model
            ModelLaunchSettings(
                MISTRAL_LARGE3_FP8_MODEL_PATH,
                tp_size=8,
                extra_args=base_args + eagle_args,
                variant="TP8+MTP",
            ),
            # Variant: "nvfp4" - NVFP4 model + TP=8 + trtllm_mla backend
            ModelLaunchSettings(
                MISTRAL_LARGE3_NVFP4_MODEL_PATH,
                tp_size=8,
                extra_args=base_args,
                variant="NVFP4",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="Mistral-Large-3",
            accuracy_params=AccuracyTestParams(dataset="gsm8k", baseline_accuracy=0.85),
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_mistral_large3",
            ),
        )
```
**EN:** This test method exercises mistral large3 all variants and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 mistral large3 all variants 场景，并验证观测到的行为是否符合预期契约。

### Lines 97-98: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `os`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `run_eval`, `unittest.main`
