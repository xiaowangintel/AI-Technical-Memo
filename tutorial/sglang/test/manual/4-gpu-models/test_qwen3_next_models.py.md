# test_qwen3_next_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/4-gpu-models/test_qwen3_next_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `qwen3 next models` scenario in `test/manual/4-gpu-models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/4-gpu-models` 中的 `qwen3 next models` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.kits.kl_divergence_kit import KLDivergenceMixin
from sglang.test.kits.prefix_cache_branching_kit import PrefixCacheBranchingMixin
from sglang.test.server_fixtures.default_fixture import DefaultServerBase

QWEN3_NEXT_MODEL = "Qwen/Qwen3-Next-80B-A3B-Instruct"
```
**EN:** This range imports `unittest`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.kl_divergence_kit` and `sglang.test.kits.prefix_cache_branching_kit`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 11-27: Class definition for TestQwen3Next / 类定义
```python
class TestQwen3Next(
    GSM8KMixin, KLDivergenceMixin, PrefixCacheBranchingMixin, DefaultServerBase
):
    model = QWEN3_NEXT_MODEL
    cache_chunk_size = 64
    gsm8k_accuracy_thres = 0.93
    kl_div_thres = 0.0025
    other_args = [
        "--tp-size",
        "4",
        "--chunked-prefill-size",
        "2048",
        "--mamba-scheduler-strategy",
        "extra_buffer",
        "--mamba-track-interval",
        "128",
    ]
```
**EN:** This range declares `TestQwen3Next`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 28-31: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Disaggregated serving / 解耦式服务
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.kl_divergence_kit`, `sglang.test.kits.prefix_cache_branching_kit`, `sglang.test.server_fixtures.default_fixture`
