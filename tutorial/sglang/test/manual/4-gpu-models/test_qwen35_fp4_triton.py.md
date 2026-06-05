# test_qwen35_fp4_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/4-gpu-models/test_qwen35_fp4_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `qwen35 fp4 triton` scenario in `test/manual/4-gpu-models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/4-gpu-models` 中的 `qwen35 fp4 triton` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Constants and scenario settings / 常量与场景配置
```python
import unittest

from sglang.test.accuracy_test_runner import AccuracyTestParams

# This eval harness applies the chat_template, which is critical for qwen3.5
# to get good accuracy on gsm8k
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.test_utils import (
    CustomTestCase,
    ModelLaunchSettings,
)

QWEN35_FP4_MODEL = "nvidia/Qwen3.5-397B-A17B-NVFP4"
ACC_THRESHOLDS = {QWEN35_FP4_MODEL: {"gsm8k": 0.95}}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. It delegates execution to `run_combined_tests`, which bundles launch settings with shared validation passes. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 它会调用 `run_combined_tests`，把启动配置与共享校验流程组合在一起。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 17-17: Class definition for TestQwen35FP4 / 类定义
```python
class TestQwen35FP4(CustomTestCase):
```
**EN:** This range declares `TestQwen35FP4`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 18-41: Test routines around test_gsm8k / 测试例程
```python
    def test_gsm8k(self):
        base_args = [
            "--tp-size",
            "4",
            "--chunked-prefill-size",
            "2048",
            "--mamba-scheduler-strategy",
            "extra_buffer",
            "--mamba-track-interval",
            "128",
            "--mamba-ssm-dtype",
            "bfloat16",
            "--max-running-requests",
            "128",
            "--reasoning-parser",
            "qwen3",
            "--attention-backend",
            "trtllm_mha",
            "--quantization",
            "modelopt_fp4",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true,"num_threads": 64}',
        ]
```
**EN:** This range defines concrete test routine(s) `test_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 42-42: Scenario logic / 场景逻辑
```python
        variants = [
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 43-66: Scenario logic / 场景逻辑
```python
            ModelLaunchSettings(
                QWEN35_FP4_MODEL,
                extra_args=base_args,
                variant="Triton",
            ),
            # TODO: Fix this and re-enable it
            # ModelLaunchSettings(
            #     QWEN35_FP4_MODEL,
            #     extra_args=base_args + ["--linear-attn-decode-backend", "flashinfer"],
            #     variant="FlashInfer",
            # ),
        ]

        run_combined_tests(
            models=variants,
            test_name="Qwen3.5-397B-A17B-NVFP4",
            accuracy_params=AccuracyTestParams(
                dataset="gsm8k",
                baseline_accuracy=ACC_THRESHOLDS[QWEN35_FP4_MODEL]["gsm8k"],
                num_examples=200,
                num_threads=128,
                max_tokens=16000,
                thinking_mode="qwen3",
                temperature=0.6,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It delegates execution to `run_combined_tests`, which bundles launch settings with shared validation passes. The code also ties this block to evaluation or benchmark execution. Representative call sites include `ModelLaunchSettings`, `run_combined_tests` and `AccuracyTestParams`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会调用 `run_combined_tests`，把启动配置与共享校验流程组合在一起。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 67-67: Scenario logic / 场景逻辑
```python
                top_p=0.95,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 68-70: Scenario logic / 场景逻辑
```python
                top_k=20,
            ),
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 71-74: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.accuracy_test_runner`, `sglang.test.run_combined_tests`, `sglang.test.test_utils`
