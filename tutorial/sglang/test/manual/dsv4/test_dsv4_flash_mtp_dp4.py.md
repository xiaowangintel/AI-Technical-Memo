# test_dsv4_flash_mtp_dp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_dsv4_flash_mtp_dp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `dsv4 flash mtp dp4` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `dsv4 flash mtp dp4` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-54: Constants and scenario settings / 常量与场景配置
```python
"""DSV4 Flash MTP test using EAGLE speculative algorithm.

DSV4 Flash MTP shares the EAGLE wire path: EAGLE algo + NextN head built
into the target model weights. No separate draft model is needed (sglang
auto-falls back `--speculative-draft-model-path` to the target model).

Test matrix mirrors test_eagle_infer_b.TestEAGLEServerBasic to maximize
cuda-graph + buffer-pool coverage on the DSV4 path:
  - test_gsm8k         (accuracy + spec path full forward)
  - test_max_token_one (degenerate spec step, still cuda-graph captured)
  - test_request_abort (cuda-graph buffer pool survives abort+restart)

Server launch matches `run_flash_dp4.sh`: tp=4, dp=4, deepep MoE backend,
DSV4 FP8 (FP4 experts disabled).
"""

import random
import threading
import time
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.few_shot_gsm8k import run_eval as run_gsm8k_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

DSV4_FLASH_MODEL_PATH = "sgl-project/DeepSeek-V4-Flash-FP8"

DSV4_FLASH_ENV = {
    "SGLANG_DSV4_FP4_EXPERTS": "0",
    # MTP runs ~num_draft_tokens forward passes per step, so the deepep
    # dispatch input size scales by that factor. Default 256 (used by the
    # plain server) overflows once cuda-graph-max-bs * num_draft_tokens
    # > 256. 1024 covers bs=128 * 4 draft tokens with headroom.
    "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "1024",
}

DEEPEP_CONFIG = '{"normal_dispatch":{"num_sms":96},"normal_combine":{"num_sms":96}}'

PROMPTS = [
    "[INST] You are a helpful assistant.\\nWhere are you from? [/INST]",
    "[INST] You are a helpful assistant.\\nSummarize gradient descent in 2 sentences. [/INST]",
    "[INST] You are a helpful assistant.\\nWhat is 17*23? [/INST]",
    "[INST] You are a helpful assistant.\\nList three primary colors. [/INST]",
]
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `needed`, `test_gsm8k`, `test_max_token_one` and `test_request_abort`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 55-56: Class definition for DSV4FlashMTPServerBase / 类定义
```python
class DSV4FlashMTPServerBase(CustomTestCase):
    @classmethod
```
**EN:** This range declares `DSV4FlashMTPServerBase`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 57-79: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DSV4_FLASH_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--tp",
            "4",
            "--dp",
            "4",
            "--enable-dp-attention",
            "--moe-a2a-backend",
            "deepep",
            "--cuda-graph-max-bs",
            "128",
            "--max-running-requests",
            "256",
            "--deepep-config",
            DEEPEP_CONFIG,
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-num-steps",
            "3",
            "--speculative-eagle-topk",
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 80-81: Scenario logic / 场景逻辑
```python
            "1",
            "--speculative-num-draft-tokens",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 82-92: Process control logic / 进程控制逻辑
```python
            "4",
            "--mem-fraction-static",
            "0.7",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
            env=DSV4_FLASH_ENV,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 93-94: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 95-96: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 98-104: Helper routines around send_request / 辅助例程
```python
    def send_request(self):
        time.sleep(random.uniform(0, 2))
        for prompt in PROMPTS:
            resp = requests.post(
                self.base_url + "/generate",
                json={
                    "text": prompt,
```
**EN:** This range implements helper routine(s) `send_request` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `sleep`, `uniform` and `post`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 105-108: Assertions and result checks / 断言与结果检查
```python
                    "sampling_params": {"temperature": 0, "max_new_tokens": 256},
                },
            )
            assert resp.status_code == 200
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。

### Lines 110-123: Helper routines around send_requests_abort / 辅助例程
```python
    def send_requests_abort(self):
        for prompt in PROMPTS:
            try:
                time.sleep(random.uniform(0, 2))
                requests.post(
                    self.base_url + "/generate",
                    json={
                        "text": prompt,
                        "sampling_params": {"temperature": 0, "max_new_tokens": 256},
                    },
                    timeout=0.5,
                )
            except requests.exceptions.Timeout:
                pass
```
**EN:** This range implements helper routine(s) `send_requests_abort` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `sleep`, `uniform` and `post`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 126-126: Class definition for TestDSV4FlashMTPBasic / 类定义
```python
class TestDSV4FlashMTPBasic(DSV4FlashMTPServerBase):
```
**EN:** This range declares `TestDSV4FlashMTPBasic`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 127-141: Test routines around test_gsm8k / 测试例程
```python
    def test_gsm8k(self):
        """Accuracy + spec path full forward."""
        requests.get(self.base_url + "/flush_cache")
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_gsm8k_eval(args)
        print(f"{metrics=}")
        self.assertGreater(metrics["accuracy"], 0.95)
```
**EN:** This range defines concrete test routine(s) `test_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `SimpleNamespace`, `split` and `run_gsm8k_eval`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 143-150: Test routines around test_max_token_one / 测试例程
```python
    def test_max_token_one(self):
        """Degenerate spec step (still cuda-graph captured)."""
        requests.get(self.base_url + "/flush_cache")
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=100,
            max_new_tokens=1,
```
**EN:** This range defines concrete test routine(s) `test_max_token_one`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `step`, `get` and `SimpleNamespace`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 151-156: Assertions and result checks / 断言与结果检查
```python
            parallel=128,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_gsm8k_eval(args)
        self.assertGreater(metrics["output_throughput"], 50)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `split`, `run_gsm8k_eval` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 158-171: Test routines around test_request_abort / 测试例程
```python
    def test_request_abort(self):
        """Cuda-graph buffer pool must survive abort+restart cycles."""
        concurrency = 4
        threads = [
            threading.Thread(target=self.send_request) for _ in range(concurrency)
        ] + [
            threading.Thread(target=self.send_requests_abort)
            for _ in range(concurrency)
        ]
        for t in threads:
            t.start()
        for t in threads:
            t.join()
        self.assertIsNone(self.process.poll())
```
**EN:** This range defines concrete test routine(s) `test_request_abort`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `Thread`, `start`, `join` and `assertIsNone`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 172-175: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `random`, `threading`, `time`, `types`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
