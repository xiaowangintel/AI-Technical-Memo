# test_mimo_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_mimo_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models mimo models in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 8 gpu models mimo models 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Import dependencies
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.kits.spec_decoding_kit import SpecDecodingMixin
from sglang.test.server_fixtures.default_fixture import DefaultServerBase
from sglang.test.server_fixtures.mmmu_fixture import MMMUServerBase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 9-9: Register CI metadata
```python
register_cuda_ci(est_time=500, stage="base-c", runner_config="8-gpu-h200")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 12-12: Define class TestMiMoV2Flash
```python
class TestMiMoV2Flash(GSM8KMixin, SpecDecodingMixin, DefaultServerBase):
```
**EN:** This declaration introduces the `TestMiMoV2Flash` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMiMoV2Flash` 测试类，并说明它通过继承承担的职责。

### Lines 13-49: Declare TestMiMoV2Flash configuration
```python
    gsm8k_accuracy_thres = 0.75
    gsm8k_num_questions = 1319
    gsm8k_num_threads = 1319
    model = "XiaomiMiMo/MiMo-V2-Flash"

    other_args = [
        "--tp",
        "4",
        "--dp",
        "2",
        "--enable-dp-attention",
        "--trust-remote-code",
        "--attention-backend",
        "fa3",
        "--max-running-requests",
        "128",
        "--cuda-graph-max-bs",
        "64",
        "--page-size",
        "64",
        "--mem-fraction-static",
        "0.75",
        "--speculative-algorithm",
        "EAGLE",
        "--speculative-num-steps",
        "3",
        "--speculative-eagle-topk",
        "1",
        "--speculative-num-draft-tokens",
        "4",
        "--enable-multi-layer-eagle",
        "--model-loader-extra-config",
        '{"enable_multithread_load": true,"num_threads": 64}',
    ]

    bs_1_speed_thres = 170
    accept_length_thres = 3.2
```
**EN:** This block defines class-level settings that are shared across the `TestMiMoV2Flash` test methods.
**CN:** 该代码块定义了 `TestMiMoV2Flash` 各测试方法共享的类级配置。

### Lines 52-77: Define module constants
```python
MIMO_V2_MODEL = "XiaomiMiMo/MiMo-V2.5"
MIMO_V2_OTHER_ARGS = [
    "--tp",
    "8",
    "--dp",
    "2",
    "--enable-dp-attention",
    "--mm-enable-dp-encoder",
    "--attention-backend",
    "fa3",
    "--mm-attention-backend",
    "fa3",
    "--reasoning-parser",
    "mimo",
]
MIMO_V2_MTP_OTHER_ARGS = MIMO_V2_OTHER_ARGS + [
    "--speculative-algorithm",
    "EAGLE",
    "--speculative-num-steps",
    "3",
    "--speculative-eagle-topk",
    "1",
    "--speculative-num-draft-tokens",
    "4",
    "--enable-multi-layer-eagle",
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 80-80: Define class TestMiMoV2
```python
class TestMiMoV2(GSM8KMixin, MMMUServerBase):
```
**EN:** This declaration introduces the `TestMiMoV2` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMiMoV2` 测试类，并说明它通过继承承担的职责。

### Lines 81-86: Declare TestMiMoV2 configuration
```python
    gsm8k_accuracy_thres = 0.75
    gsm8k_accept_length_thres = 2.5
    model = MIMO_V2_MODEL
    mem_fraction_static = 0.65
    server_api_key = None
    other_args = MIMO_V2_MTP_OTHER_ARGS
```
**EN:** This block defines class-level settings that are shared across the `TestMiMoV2` test methods.
**CN:** 该代码块定义了 `TestMiMoV2` 各测试方法共享的类级配置。

### Lines 89-90: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.spec_decoding_kit`, `sglang.test.server_fixtures.default_fixture`, `sglang.test.server_fixtures.mmmu_fixture`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
