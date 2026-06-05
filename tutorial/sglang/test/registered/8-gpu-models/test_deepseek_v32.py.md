# test_deepseek_v32.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_deepseek_v32.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models deepseek v32 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 8 gpu models deepseek v32 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies
```python
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.performance_test_runner import PerformanceTestParams
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import ModelLaunchSettings, is_blackwell_system
from sglang.test.tool_call_test_runner import ToolCallTestParams
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 10-10: Register CI metadata
```python
register_cuda_ci(est_time=5400, suite="nightly-8-gpu-common", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 12-29: Define module constants
```python
DEEPSEEK_V32_MODEL_PATH = "deepseek-ai/DeepSeek-V3.2"

BASE_ARGS = [
    "--trust-remote-code",
    "--model-loader-extra-config",
    '{"enable_multithread_load": true}',
]

TOOL_CALL_ARGS = [
    "--tool-call-parser=deepseekv32",
    "--reasoning-parser=deepseek-v3",
]

DP_ARGS = [
    "--tp=8",
    "--dp=8",
    "--enable-dp-attention",
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 32-33: Define module constants
```python
GSM8K_BASELINE = 0.935
GPQA_BASELINE = 0.83
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 36-36: Define class TestDeepseekV32
```python
class TestDeepseekV32(unittest.TestCase):
```
**EN:** This declaration introduces the `TestDeepseekV32` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepseekV32` 测试类，并说明它通过继承承担的职责。

### Lines 37-44: Document the class `TestDeepseekV32`
```python
    """Unified test class for DeepSeek V3.2 performance and accuracy.

    Tests multiple variants with both performance and accuracy tests:
    - dp: Standard TP=8 + DP=8 with dp-attention
    - dp+mtp: DP + EAGLE speculative decoding
    - tp: Pure TP=8 only
    - tp+mtp: Pure TP=8 + EAGLE speculative decoding
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDeepseekV32`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDeepseekV32`的设计意图。

### Lines 46-102: Run test: deepseek v32 all variants
```python
    def test_deepseek_v32_all_variants(self):
        """Run performance and accuracy for all DeepSeek V3.2 variants."""
        TP_ARGS = [
            "--tp=8",
        ]
        MTP_ARGS = [
            "--speculative-algorithm=EAGLE",
            "--speculative-num-steps=3",
            "--speculative-eagle-topk=1",
            "--speculative-num-draft-tokens=4",
            "--mem-frac=0.7",
        ]
        variants = [
            # Variant: "dp" - Standard TP=8 + DP=8 with dp-attention
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + DP_ARGS + TOOL_CALL_ARGS,
                variant="DP8",
            ),
            # Variant: "dp+mtp" - DP + EAGLE speculative decoding
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + DP_ARGS + TOOL_CALL_ARGS + MTP_ARGS,
                variant="DP8+MTP",
            ),
            # Variant: "tp" - Pure TP=8 only
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + TP_ARGS + TOOL_CALL_ARGS,
                variant="TP8",
            ),
            # Variant: "tp+mtp" - Pure TP=8 + EAGLE speculative decoding
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + TP_ARGS + TOOL_CALL_ARGS + MTP_ARGS,
                variant="TP8+MTP",
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="DeepSeek-V3.2",
            accuracy_params=AccuracyTestParams(
                dataset="gsm8k", baseline_accuracy=GSM8K_BASELINE
            ),
            performance_params=PerformanceTestParams(
                batch_sizes=[1, 8, 16, 64],
                profile_dir="performance_profiles_deepseek_v32",
            ),
            tool_call_params=ToolCallTestParams(
                test_thinking=True, test_reasoning_usage=True
            ),
        )
```
**EN:** This test method exercises deepseek v32 all variants and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 deepseek v32 all variants 场景，并验证观测到的行为是否符合预期契约。

### Lines 104-158: Run test: deepseek v32 nsa backends
```python
    @unittest.skipIf(is_blackwell_system(), "Requires H200 system")
    def test_deepseek_v32_nsa_backends(self):
        """Test NSA attention backend variants (H200 only).

        Tests three NSA backend configurations:
        - flashmla: flashmla_sparse prefill + flashmla_kv decode
        - fa3: FA3 prefill + FA3 decode
        - fp8kvcache: default backends with FP8 KV cache
        """
        NSA_FLASHMLA_ARGS = [
            "--attention-backend=nsa",
            "--nsa-prefill-backend=flashmla_sparse",
            "--nsa-decode-backend=flashmla_kv",
        ]

        NSA_FA3_ARGS = [
            "--attention-backend=nsa",
            "--nsa-prefill-backend=fa3",
            "--nsa-decode-backend=fa3",
        ]

        NSA_FP8KV_ARGS = [
            "--attention-backend=nsa",
            "--kv-cache-dtype=fp8_e4m3",
        ]

        nsa_variants = [
            # flashmla backend
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + DP_ARGS + NSA_FLASHMLA_ARGS,
            ),
            # fa3 backend
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + DP_ARGS + NSA_FA3_ARGS,
            ),
            # fp8 kv cache
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + DP_ARGS + NSA_FP8KV_ARGS,
            ),
        ]

        run_combined_tests(
            models=nsa_variants,
            test_name="DeepSeek-V3.2 NSA Backends",
            accuracy_params=AccuracyTestParams(
                dataset="gsm8k", baseline_accuracy=GSM8K_BASELINE
            ),
            performance_params=None,
        )
```
**EN:** This test method exercises deepseek v32 nsa backends and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 deepseek v32 nsa backends 场景，并验证观测到的行为是否符合预期契约。

### Lines 160-196: Run test: deepseek v32 b200
```python
    @unittest.skipIf(
        not is_blackwell_system(),
        "Hardware agnostic - just using B200 for efficiency reasons",
    )
    def test_deepseek_v32_b200(self):
        """Test DeepSeek V3.2 with GPQA evaluation using thinking mode (B200 only).

        This test runs GPQA evaluation with the reasoning parser enabled.
        """
        B200_REASONING_ARGS = [
            "--tool-call-parser=deepseekv32",
            "--reasoning-parser=deepseek-v3",
        ]

        variants = [
            ModelLaunchSettings(
                DEEPSEEK_V32_MODEL_PATH,
                tp_size=8,
                extra_args=BASE_ARGS + DP_ARGS + B200_REASONING_ARGS,
            ),
        ]

        run_combined_tests(
            models=variants,
            test_name="DeepSeek-V3.2 GPQA (B200)",
            accuracy_params=AccuracyTestParams(
                dataset="gpqa",
                baseline_accuracy=GPQA_BASELINE,
                num_examples=198,
                num_threads=198,
                max_tokens=120000,
                thinking_mode="deepseek-v3",
                temperature=0.1,
                repeat=4,
            ),
            performance_params=None,  # Skip performance test for GPQA
        )
```
**EN:** This test method exercises deepseek v32 b200 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 deepseek v32 b200 场景，并验证观测到的行为是否符合预期契约。

### Lines 199-200: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.accuracy_test_runner`, `sglang.test.ci.ci_register`, `sglang.test.performance_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`, `sglang.test.tool_call_test_runner`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
