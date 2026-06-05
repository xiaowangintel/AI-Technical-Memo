# test_gpt_oss_120b.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_gpt_oss_120b.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models gpt oss 120b in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 8 gpu models gpt oss 120b 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Import dependencies
```python
import unittest

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

### Lines 12-13: Define module constants
```python
GPT_OSS_120B_MXFP4_MODEL_PATH = "openai/gpt-oss-120b"
GPT_OSS_120B_EAGLE3_DRAFT_MODEL_PATH = "lmsys/EAGLE3-gpt-oss-120b-bf16"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 16-16: Define class TestGptOss120B
```python
class TestGptOss120B(unittest.TestCase):
```
**EN:** This declaration introduces the `TestGptOss120B` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGptOss120B` 测试类，并说明它通过继承承担的职责。

### Lines 17-22: Document the class `TestGptOss120B`
```python
    """Unified test class for GPT-OSS-120B performance and accuracy.

    Testing:
    - Basic configs for MXFP4
    - Full config for MXFP4 with reasoning-parser, tool-call-parser, and MTP
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGptOss120B`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGptOss120B`的设计意图。

### Lines 24-79: Run test: gpt oss 120b all variants
```python
    def test_gpt_oss_120b_all_variants(self):
        """Run performance and accuracy for all GPT-OSS-120B variants."""
        base_args = [
            "--tp=8",
            "--trust-remote-code",
            "--cuda-graph-max-bs=200",
            "--mem-fraction-static=0.93",
        ]
        # Lower batch size for EAGLE3 variants to avoid OOM
        base_args_eagle3 = [
            "--tp=8",
            "--trust-remote-code",
            "--cuda-graph-max-bs=100",
            "--mem-fraction-static=0.85",
        ]
        parser_args = [
            "--reasoning-parser=gpt-oss",
            "--tool-call-parser=gpt-oss",
        ]
        eagle3_args = [
            "--speculative-algorithm=EAGLE3",
            f"--speculative-draft-model-path={GPT_OSS_120B_EAGLE3_DRAFT_MODEL_PATH}",
            "--speculative-num-steps=3",
            "--speculative-eagle-topk=1",
            "--speculative-num-draft-tokens=4",
        ]
        eagle3_env = {
            "SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN": "1",
        }

        variants = [
            # Variant 1: MXFP4 baseline
            ModelLaunchSettings(
                GPT_OSS_120B_MXFP4_MODEL_PATH,
                tp_size=8,
                extra_args=base_args,
                variant="MXFP4",
            ),
            # Variant 2: MXFP4 + Parsers + EAGLE3 (full featured quantized, lower batch size)
            ModelLaunchSettings(
                GPT_OSS_120B_MXFP4_MODEL_PATH,
                tp_size=8,
                extra_args=base_args_eagle3 + parser_args + eagle3_args,
                env=eagle3_env,
                variant="MXFP4+Parsers+EAGLE3",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="GPT-OSS-120B",
            accuracy_params=None,
            performance_params=PerformanceTestParams(
                profile_dir="performance_profiles_gpt_oss_120b",
            ),
        )
```
**EN:** This test method exercises gpt oss 120b all variants and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 gpt oss 120b all variants 场景，并验证观测到的行为是否符合预期契约。

### Lines 82-83: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
