# test_gb300_pro.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_gb300_pro.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `gb300 pro` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `gb300 pro` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Constants and scenario settings / 常量与场景配置
```python
"""GB300 x DeepSeek-V4-Pro.

Single-node TP=4 path. Note that GB300 Pro CP bumps
mem-fraction-static to 0.88 (1.6T weights at TP=4 on 273 GB don't
fit at the default 0.78). Covers Low-Latency, Balanced,
Max-Throughput, Context-Parallel (CP).
"""

import os
import sys
import unittest

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
from _common import DEEPEP_LARGE_SMS_CONFIG, DSV4ProAime25TestBase

MODEL = "deepseek-ai/DeepSeek-V4-Pro"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `Parallel`, `insert`, `dirname` and `abspath`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 19-41: Class definition for TestGB300ProLowLatency / 类定义
```python
class TestGB300ProLowLatency(DSV4ProAime25TestBase):
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
        "--mem-fraction-static",
        "0.88",
    ]
    EXTRA_ENV = {}
```
**EN:** This range declares `TestGB300ProLowLatency`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 44-68: Class definition for TestGB300ProBalanced / 类定义
```python
class TestGB300ProBalanced(DSV4ProAime25TestBase):
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
        "--mem-fraction-static",
        "0.9",
        "--cuda-graph-max-bs",
        "128",
        "--max-running-requests",
        "256",
```
**EN:** This range declares `TestGB300ProBalanced`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 69-72: Constants and scenario settings / 常量与场景配置
```python
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {"SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "256"}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 75-95: Class definition for TestGB300ProMaxThroughput / 类定义
```python
class TestGB300ProMaxThroughput(DSV4ProAime25TestBase):
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
        "--mem-fraction-static",
        "0.9",
        "--cuda-graph-max-bs",
        "128",
        "--max-running-requests",
        "256",
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {"SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "256"}
```
**EN:** This range declares `TestGB300ProMaxThroughput`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 98-122: Class definition for TestGB300ProCP / 类定义
```python
class TestGB300ProCP(DSV4ProAime25TestBase):
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
        "0.88",
        "--cuda-graph-max-bs",
        "256",
        "--max-running-requests",
        "256",
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {
        "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "256",
    }
```
**EN:** This range declares `TestGB300ProCP`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 123-126: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `sys`, `unittest`
- **Third-party / 第三方库**: `_common`
- **Project Modules / 项目模块**: None / 无
