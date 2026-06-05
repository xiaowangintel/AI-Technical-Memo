# test_dsv4_flash_sanity_dp4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_dsv4_flash_sanity_dp4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `dsv4 flash sanity dp4` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `dsv4 flash sanity dp4` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Constants and scenario settings / 常量与场景配置
```python
"""DSV4-Flash 4-GPU server sanity matrix (TP4 variants)."""

import unittest

from sglang.srt.utils import kill_process_tree
from sglang.test.kits.server_sanity_kit import ServerSanityMixin
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

DSV4_FLASH_MODEL_PATH = "sgl-project/DeepSeek-V4-Flash-FP8"

DSV4_FLASH_ENV = {
    "SGLANG_DSV4_FP4_EXPERTS": "0",
    "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "1024",
}

DEEPEP_CONFIG = '{"normal_dispatch":{"num_sms":96},"normal_combine":{"num_sms":96}}'
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `matrix`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 24-34: Helper routines around _launch / 辅助例程
```python
def _launch(other_args, env_extra=None, timeout_mult=1):
    env = dict(DSV4_FLASH_ENV)
    if env_extra:
        env.update(env_extra)
    return popen_launch_server(
        DSV4_FLASH_MODEL_PATH,
        DEFAULT_URL_FOR_TEST,
        timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * timeout_mult,
        other_args=other_args,
        env=env,
    )
```
**EN:** This range implements helper routine(s) `_launch` so setup, transformation, or validation logic can be reused cleanly. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `update` and `popen_launch_server`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 35-48: Scenario logic / 场景逻辑
```python


_EAGLE_SPEC_ARGS = [
    "--speculative-algorithm",
    "EAGLE",
    "--speculative-num-steps",
    "3",
    "--speculative-eagle-topk",
    "1",
    "--speculative-num-draft-tokens",
    "4",
]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 49-52: Class definition for TestDSV4FlashTP4DP4 / 类定义
```python
class TestDSV4FlashTP4DP4(ServerSanityMixin, CustomTestCase):
    """TP4 + DP4 + deepep + EAGLE MTP."""

    @classmethod
```
**EN:** This range declares `TestDSV4FlashTP4DP4`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 53-73: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = _launch(
            [
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
                "--mem-fraction-static",
                "0.7",
                *_EAGLE_SPEC_ARGS,
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `_launch`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 74-75: Scenario logic / 场景逻辑
```python
            ]
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 76-77: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 78-79: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-85: Class definition for TestDSV4FlashTP4EP / 类定义
```python
class TestDSV4FlashTP4EP(ServerSanityMixin, CustomTestCase):
    """TP attn + EP MoE (no DP attn) — exercises the DeepEP + TP-attn path."""

    @classmethod
```
**EN:** This range declares `TestDSV4FlashTP4EP`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `MoE`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 86-106: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = _launch(
            [
                "--trust-remote-code",
                "--tp",
                "4",
                "--ep",
                "4",
                # No --enable-dp-attention by design: covers TP-attn path.
                "--moe-a2a-backend",
                "deepep",
                "--cuda-graph-max-bs",
                "128",
                "--max-running-requests",
                "64",
                "--deepep-config",
                DEEPEP_CONFIG,
                "--mem-fraction-static",
                "0.7",
                *_EAGLE_SPEC_ARGS,
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `_launch`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 107-108: Scenario logic / 场景逻辑
```python
            ]
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 109-110: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 111-112: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 115-118: Class definition for TestDSV4FlashTP4DP4ChunkedPrefillLarge / 类定义
```python
class TestDSV4FlashTP4DP4ChunkedPrefillLarge(ServerSanityMixin, CustomTestCase):
    """TP4 + DP4 with --chunked-prefill-size 16384 — large chunked prefill."""

    @classmethod
```
**EN:** This range declares `TestDSV4FlashTP4DP4ChunkedPrefillLarge`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 119-139: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = _launch(
            [
                "--trust-remote-code",
                "--tp",
                "4",
                "--dp",
                "4",
                "--enable-dp-attention",
                "--moe-a2a-backend",
                "deepep",
                "--chunked-prefill-size",
                "16384",
                "--cuda-graph-max-bs",
                "128",
                "--max-running-requests",
                "256",
                "--deepep-config",
                DEEPEP_CONFIG,
                "--mem-fraction-static",
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `_launch`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 140-143: Scenario logic / 场景逻辑
```python
                "0.7",
                *_EAGLE_SPEC_ARGS,
            ]
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 144-145: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 146-147: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 148-151: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.kits.server_sanity_kit`, `sglang.test.test_utils`
