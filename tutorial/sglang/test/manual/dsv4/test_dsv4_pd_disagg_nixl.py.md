# test_dsv4_pd_disagg_nixl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_dsv4_pd_disagg_nixl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `dsv4 pd disagg nixl` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `dsv4 pd disagg nixl` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Constants and scenario settings / 常量与场景配置
```python
"""DSV4 Flash PD-disagg with NIXL backend. Both sides run dp-attention
+ deepep + EAGLE MTP so attn_tp_size and the V4 state pool layout are
fully symmetric: same SWA item_len under matching attn_tp, and same
NSA c4/c128 indexer ring buffer size under matching spec status. nixl
`send_state` is page-by-index and has no V4 TP-slice / spec-asymmetric
path, so any layout mismatch would trip the item_len assert in
`nixl/conn.py`."""

import unittest
from types import SimpleNamespace

from sglang.test.few_shot_gsm8k import run_eval as run_gsm8k_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
)

DSV4_FLASH_MODEL_PATH = "sgl-project/DeepSeek-V4-Flash-FP8"

DSV4_FLASH_ENV = {
    "SGLANG_DSV4_FP4_EXPERTS": "0",
    # MTP num_draft_tokens=4 scales dispatch by ~4x; 256 overflows at bs=128.
    "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "1024",
}

DEEPEP_CONFIG = '{"normal_dispatch":{"num_sms":96},"normal_combine":{"num_sms":96}}'
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Process management appears here to launch, monitor, or stop runtime components. Assertions in this block enforce the intended outcome.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中的断言会落实预期结果。

### Lines 32-33: Class definition for TestDSV4FlashPDDisaggNIXL / 类定义
```python
class TestDSV4FlashPDDisaggNIXL(PDDisaggregationServerBase):
    @classmethod
```
**EN:** This range declares `TestDSV4FlashPDDisaggNIXL`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 34-45: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        super().setUpClass()
        cls.transfer_backend = ["--disaggregation-transfer-backend", "nixl"]
        cls.rdma_devices = []
        cls.model = DSV4_FLASH_MODEL_PATH

        cls.start_prefill()
        cls.start_decode()

        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)
        cls.launch_lb()
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `start_prefill`, `start_decode`, `wait_server_ready` and `launch_lb`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 46-47: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 48-56: Helper routines around start_prefill / 辅助例程
```python
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--base-gpu-id",
            "0",
            "--tp",
            "4",
```
**EN:** This range implements helper routine(s) `start_prefill` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 57-72: Scenario logic / 场景逻辑
```python
            "--dp",
            "4",
            "--enable-dp-attention",
            "--moe-a2a-backend",
            "deepep",
            "--deepep-config",
            DEEPEP_CONFIG,
            "--cuda-graph-max-bs",
            "128",
            "--max-running-requests",
            "256",
            "--mem-fraction-static",
            "0.7",
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-num-steps",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 73-81: Process control logic / 进程控制逻辑
```python
            "3",
            "--speculative-eagle-topk",
            "1",
            "--speculative-num-draft-tokens",
            "4",
            *cls.transfer_backend,
            *cls.rdma_devices,
        ]
        cls.process_prefill = popen_launch_pd_server(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-87: Scenario logic / 场景逻辑
```python
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
            env=DSV4_FLASH_ENV,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 88-89: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 90-106: Helper routines around start_decode / 辅助例程
```python
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--base-gpu-id",
            "4",
            "--tp",
            "4",
            "--dp",
            "4",
            "--enable-dp-attention",
            "--moe-a2a-backend",
            "deepep",
            "--deepep-config",
            DEEPEP_CONFIG,
            "--cuda-graph-max-bs",
```
**EN:** This range implements helper routine(s) `start_decode` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 107-114: Scenario logic / 场景逻辑
```python
            "128",
            "--max-running-requests",
            "256",
            "--mem-fraction-static",
            "0.7",
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-num-steps",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 115-129: Process control logic / 进程控制逻辑
```python
            "3",
            "--speculative-eagle-topk",
            "1",
            "--speculative-num-draft-tokens",
            "4",
            *cls.transfer_backend,
            *cls.rdma_devices,
        ]
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
            env=DSV4_FLASH_ENV,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `popen_launch_pd_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 131-131: Test routines around test_gsm8k / 测试例程
```python
    def test_gsm8k(self):
```
**EN:** This range defines concrete test routine(s) `test_gsm8k`. The logic drives the target scenario and encodes the expected acceptance criteria. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 132-144: Assertions and result checks / 断言与结果检查
```python
        """End-to-end PD-disagg accuracy through the LB."""
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=64,
            host=f"http://{self.base_host}",
            port=int(self.lb_port),
        )
        metrics = run_gsm8k_eval(args)
        print(f"{metrics=}")
        self.assertGreater(metrics["accuracy"], 0.95)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. The code also ties this block to evaluation or benchmark execution. Assertions in this block enforce the intended outcome. Representative call sites include `SimpleNamespace`, `run_gsm8k_eval` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 145-148: Script entry point / 脚本入口
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
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.few_shot_gsm8k`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
