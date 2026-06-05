# test_qwen35_models_archived.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/4-gpu-models/test_qwen35_models_archived.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `qwen35 models archived` scenario in `test/manual/4-gpu-models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/4-gpu-models` 中的 `qwen35 models archived` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Constants and scenario settings / 常量与场景配置
```python
"""Archived test classes split out of test/registered/4-gpu-models/test_qwen35_models.py.

Originally registered with `register_cuda_ci(...)`. Moved here as part of
the per-commit pruning effort to keep the code reachable manually.
Run with `python3 test/manual/4-gpu-models/test_qwen35_models_archived.py`.
"""

import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.accuracy_test_runner import AccuracyTestParams
from sglang.test.kits.reasoning_kit import ReasoningTokenUsageMixin

# This eval harness applies the chat_template, which is critical for qwen3.5
# to get good accuracy on gsm8k
from sglang.test.run_combined_tests import run_combined_tests
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    ModelLaunchSettings,
    popen_launch_server,
)

QWEN35_FP4_MODEL = "nvidia/Qwen3.5-397B-A17B-NVFP4"
ACC_THRESHOLDS = {QWEN35_FP4_MODEL: {"gsm8k": 0.95}}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. It delegates execution to `run_combined_tests`, which bundles launch settings with shared validation passes. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `register_cuda_ci`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 它会调用 `run_combined_tests`，把启动配置与共享校验流程组合在一起。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 33-33: Class definition for TestQwen35FP4 / 类定义
```python
class TestQwen35FP4(CustomTestCase):
```
**EN:** This range declares `TestQwen35FP4`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 34-57: Test routines around test_gsm8k / 测试例程
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

### Lines 58-58: Scenario logic / 场景逻辑
```python
        variants = [
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 59-82: Scenario logic / 场景逻辑
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

### Lines 83-83: Scenario logic / 场景逻辑
```python
                top_p=0.95,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 84-86: Scenario logic / 场景逻辑
```python
                top_k=20,
            ),
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 89-92: Class definition for TestQwen35FP4MTP / 类定义
```python
class TestQwen35FP4MTP(ReasoningTokenUsageMixin, CustomTestCase):
    reasoning_parser_name = "qwen3"

    @classmethod
```
**EN:** This range declares `TestQwen35FP4MTP`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 93-113: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = QWEN35_FP4_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.init_reasoning_token_verifier()
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
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
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `init_reasoning_token_verifier` and `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 114-117: Scenario logic / 场景逻辑
```python
                "--reasoning-parser",
                "qwen3",
                "--attention-backend",
                "trtllm_mha",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 118-133: Scenario logic / 场景逻辑
```python
                "--quantization",
                "modelopt_fp4",
                "--speculative-algorithm",
                "NEXTN",
                "--speculative-num-steps",
                "3",
                "--speculative-eagle-topk",
                "1",
                "--speculative-num-draft-tokens",
                "4",
                "--mem-fraction-static",
                "0.8",
                "--model-loader-extra-config",
                '{"enable_multithread_load": true,"num_threads": 64}',
            ],
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 134-135: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 136-137: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 139-163: Test routines around test_gsm8k / 测试例程
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            model=self.model,
            eval_name="gsm8k",
            num_shots=5,
            num_examples=200,
            max_tokens=16000,
            num_threads=128,
            repeat=1,
            temperature=0.6,
            top_p=0.95,
            top_k=20,
            base_url=self.base_url,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval(args)
        print(f"{metrics=}")
        self.assertGreaterEqual(metrics["score"], ACC_THRESHOLDS[self.model]["gsm8k"])

        server_info = requests.get(self.base_url + "/server_info")
        avg_spec_accept_length = server_info.json()["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
```
**EN:** This range defines concrete test routine(s) `test_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `SimpleNamespace`, `split`, `run_eval` and `assertGreaterEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 164-164: Assertions and result checks / 断言与结果检查
```python
        self.assertGreater(avg_spec_accept_length, 3.3)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 165-168: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `types`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.accuracy_test_runner`, `sglang.test.kits.reasoning_kit`, `sglang.test.run_combined_tests`, `sglang.test.run_eval`, `sglang.test.test_utils`
