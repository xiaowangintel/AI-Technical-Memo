# test_qwen3_next_models_mtp_archived.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/4-gpu-models/test_qwen3_next_models_mtp_archived.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `qwen3 next models mtp archived` scenario in `test/manual/4-gpu-models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/4-gpu-models` 中的 `qwen3 next models mtp archived` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Constants and scenario settings / 常量与场景配置
```python
"""Archived test classes split out of test/registered/4-gpu-models/test_qwen3_next_models_mtp.py.

Originally registered with `register_cuda_ci(...)`. Moved here as part of
the per-commit pruning effort to keep the code reachable manually.
Run with `python3 test/manual/4-gpu-models/test_qwen3_next_models_mtp_archived.py`.
"""

import unittest

from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.kits.kl_divergence_kit import KLDivergenceMixin
from sglang.test.server_fixtures.default_fixture import DefaultServerBase

QWEN3_NEXT_MODEL = "Qwen/Qwen3-Next-80B-A3B-Instruct"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. The code also ties this block to evaluation or benchmark execution. Representative call sites include `register_cuda_ci`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 17-40: Class definition for TestQwen3NextMTP / 类定义
```python
class TestQwen3NextMTP(GSM8KMixin, KLDivergenceMixin, DefaultServerBase):
    model = QWEN3_NEXT_MODEL
    gsm8k_accuracy_thres = 0.93
    kl_div_thres = 0.0025
    other_args = [
        "--trust-remote-code",
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
        "--tp",
        "4",
        "--chunked-prefill-size",
        "2048",
        "--mamba-scheduler-strategy",
        "no_buffer",
        "--disable-radix-cache",
    ]
```
**EN:** This range declares `TestQwen3NextMTP`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 41-44: Script entry point / 脚本入口
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
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.kl_divergence_kit`, `sglang.test.server_fixtures.default_fixture`
