# test_b300_flash.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_b300_flash.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `b300 flash` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `b300 flash` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Constants and scenario settings / 常量与场景配置
```python
"""B300 x DeepSeek-V4-Flash.

The cookbook generator aliases B300 to B200, so the launch flags
are identical to the B200(FP4) Flash cell. Kept as a separate file
because the hardware target (and therefore the runtime environment)
is different. Covers Low-Latency, Balanced, Max-Throughput, CP.
"""

import os
import sys
import unittest

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
from _common import DEEPEP_LARGE_SMS_CONFIG, DSV4FlashAime25TestBase

MODEL = "deepseek-ai/DeepSeek-V4-Flash"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `B200`, `target`, `insert` and `dirname`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 19-39: Class definition for TestB300FlashLowLatency / 类定义
```python
class TestB300FlashLowLatency(DSV4FlashAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "4",
        "--moe-runner-backend",
        "flashinfer_mxfp4",
        "--speculative-algorithm",
        "EAGLE",
        "--speculative-num-steps",
        "3",
        "--speculative-eagle-topk",
        "1",
        "--speculative-num-draft-tokens",
        "4",
        "--chunked-prefill-size",
        "4096",
        "--disable-flashinfer-autotune",
    ]
    EXTRA_ENV = {}
```
**EN:** This range declares `TestB300FlashLowLatency`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 42-64: Class definition for TestB300FlashBalanced / 类定义
```python
class TestB300FlashBalanced(DSV4FlashAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "4",
        "--dp",
        "4",
        "--enable-dp-attention",
        "--moe-a2a-backend",
        "deepep",
        "--speculative-algorithm",
        "EAGLE",
        "--speculative-num-steps",
        "1",
        "--speculative-eagle-topk",
        "1",
        "--speculative-num-draft-tokens",
        "2",
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {"SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "1024"}
```
**EN:** This range declares `TestB300FlashBalanced`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 67-81: Class definition for TestB300FlashMaxThroughput / 类定义
```python
class TestB300FlashMaxThroughput(DSV4FlashAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "4",
        "--dp",
        "4",
        "--enable-dp-attention",
        "--moe-a2a-backend",
        "deepep",
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {"SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "1024"}
```
**EN:** This range declares `TestB300FlashMaxThroughput`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 84-106: Class definition for TestB300FlashCP / 类定义
```python
class TestB300FlashCP(DSV4FlashAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "4",
        "--moe-a2a-backend",
        "deepep",
        "--enable-nsa-prefill-context-parallel",
        "--nsa-prefill-cp-mode",
        "round-robin-split",
        "--chunked-prefill-size",
        "16384",
        "--mem-fraction-static",
        "0.78",
        "--max-running-requests",
        "1024",
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {
        "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "1024",
    }
```
**EN:** This range declares `TestB300FlashCP`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 107-110: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Quantization configuration / 量化配置
- Environment-aware configuration / 环境感知配置
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `sys`, `unittest`
- **Third-party / 第三方库**: `_common`
- **Project Modules / 项目模块**: None / 无
