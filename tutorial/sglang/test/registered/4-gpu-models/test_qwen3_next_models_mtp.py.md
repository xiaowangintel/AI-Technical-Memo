# test_qwen3_next_models_mtp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/4-gpu-models/test_qwen3_next_models_mtp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 4 gpu models qwen3 next models mtp in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 4 gpu models qwen3 next models mtp 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Import dependencies
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.kits.kl_divergence_kit import KLDivergenceMixin
from sglang.test.kits.prefix_cache_branching_kit import PrefixCacheBranchingMixin
from sglang.test.server_fixtures.default_fixture import DefaultServerBase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 9-9: Register CI metadata
```python
register_cuda_ci(est_time=290, stage="base-c", runner_config="4-gpu-h100")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 11-11: Define module constants
```python
QWEN3_NEXT_MODEL = "Qwen/Qwen3-Next-80B-A3B-Instruct"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 14-14: Define class TestQwen3NextMTPTopk
```python
class TestQwen3NextMTPTopk(
```
**EN:** This declaration introduces the `TestQwen3NextMTPTopk` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen3NextMTPTopk` 测试类，并说明它通过继承承担的职责。

### Lines 17-41: Declare TestQwen3NextMTPTopk configuration
```python
    model = QWEN3_NEXT_MODEL
    cache_chunk_size = 64
    gsm8k_accuracy_thres = 0.93
    kl_div_thres = 0.008
    other_args = [
        "--trust-remote-code",
        "--speculative-algorithm",
        "NEXTN",
        "--speculative-num-steps",
        "5",
        "--speculative-eagle-topk",
        "4",
        "--speculative-num-draft-tokens",
        "8",
        "--mem-fraction-static",
        "0.8",
        "--tp",
        "4",
        "--chunked-prefill-size",
        "2048",
        "--mamba-scheduler-strategy",
        "extra_buffer",
        "--mamba-track-interval",
        "128",
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestQwen3NextMTPTopk` test methods.
**CN:** 该代码块定义了 `TestQwen3NextMTPTopk` 各测试方法共享的类级配置。

### Lines 44-44: Define class TestQwen3NextMTPV2
```python
class TestQwen3NextMTPV2(GSM8KMixin, KLDivergenceMixin, DefaultServerBase):
```
**EN:** This declaration introduces the `TestQwen3NextMTPV2` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen3NextMTPV2` 测试类，并说明它通过继承承担的职责。

### Lines 45-68: Declare TestQwen3NextMTPV2 configuration
```python
    model = QWEN3_NEXT_MODEL
    gsm8k_accuracy_thres = 0.93
    kl_div_thres = 0.0035
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
        "extra_buffer",
        "--mamba-track-interval",
        "128",
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestQwen3NextMTPV2` test methods.
**CN:** 该代码块定义了 `TestQwen3NextMTPV2` 各测试方法共享的类级配置。

### Lines 71-72: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.kl_divergence_kit`, `sglang.test.kits.prefix_cache_branching_kit`, `sglang.test.server_fixtures.default_fixture`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
