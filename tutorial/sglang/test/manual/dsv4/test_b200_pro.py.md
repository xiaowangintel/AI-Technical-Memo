# test_b200_pro.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_b200_pro.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `b200 pro` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `b200 pro` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Constants and scenario settings / 常量与场景配置
```python
"""B200 (FP4) x DeepSeek-V4-Pro.

Covers the four cookbook recipes for this hardware x model_size cell:
Low-Latency, Balanced, Max-Throughput, Context-Parallel (CP).
"""

import os
import sys
import unittest

sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
from _common import DEEPEP_LARGE_SMS_CONFIG, DSV4ProAime25TestBase

MODEL = "deepseek-ai/DeepSeek-V4-Pro"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Representative call sites include `B200`, `Parallel`, `insert` and `dirname`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 17-39: Class definition for TestB200ProLowLatency / 类定义
```python
class TestB200ProLowLatency(DSV4ProAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "8",
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
**EN:** This range declares `TestB200ProLowLatency`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 42-66: Class definition for TestB200ProBalanced / 类定义
```python
class TestB200ProBalanced(DSV4ProAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "8",
        "--dp",
        "8",
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
        "0.82",
        "--cuda-graph-max-bs",
        "64",
        "--max-running-requests",
        "128",
```
**EN:** This range declares `TestB200ProBalanced`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 67-70: Constants and scenario settings / 常量与场景配置
```python
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {"SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "256"}
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 73-93: Class definition for TestB200ProMaxThroughput / 类定义
```python
class TestB200ProMaxThroughput(DSV4ProAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "8",
        "--dp",
        "8",
        "--enable-dp-attention",
        "--moe-a2a-backend",
        "deepep",
        "--mem-fraction-static",
        "0.82",
        "--cuda-graph-max-bs",
        "64",
        "--max-running-requests",
        "256",
        "--deepep-config",
        DEEPEP_LARGE_SMS_CONFIG,
    ]
    EXTRA_ENV = {"SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "256"}
```
**EN:** This range declares `TestB200ProMaxThroughput`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 96-120: Class definition for TestB200ProCP / 类定义
```python
class TestB200ProCP(DSV4ProAime25TestBase):
    MODEL = MODEL
    OTHER_ARGS = [
        "--trust-remote-code",
        "--tp",
        "8",
        "--moe-a2a-backend",
        "deepep",
        "--enable-nsa-prefill-context-parallel",
        "--nsa-prefill-cp-mode",
        "round-robin-split",
        "--chunked-prefill-size",
        "16384",
        "--mem-fraction-static",
        "0.78",
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
**EN:** This range declares `TestB200ProCP`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 121-124: Script entry point / 脚本入口
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
