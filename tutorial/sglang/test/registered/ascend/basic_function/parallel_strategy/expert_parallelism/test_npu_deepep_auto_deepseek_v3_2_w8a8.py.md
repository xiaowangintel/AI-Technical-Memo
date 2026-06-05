# test_npu_deepep_auto_deepseek_v3_2_w8a8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/parallel_strategy/expert_parallelism/test_npu_deepep_auto_deepseek_v3_2_w8a8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on expert parallelism npu deepep auto deepseek v3 2 w8a8 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 expert parallelism npu deepep auto deepseek v3 2 w8a8 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies
```python
import os
import unittest

from sglang.test.ascend.gsm8k_ascend_mixin import GSM8KAscendMixin
from sglang.test.ascend.test_ascend_utils import DEEPSEEK_V3_2_W8A8_WEIGHTS_PATH
from sglang.test.ascend.test_mmlu import TestMMLU
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 10-10: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-16-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 13-13: Define class TestDeepEpDeepseekV32
```python
class TestDeepEpDeepseekV32(GSM8KAscendMixin, TestMMLU, CustomTestCase):
```
**EN:** This declaration introduces the `TestDeepEpDeepseekV32` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepEpDeepseekV32` 测试类，并说明它通过继承承担的职责。

### Lines 14-19: Document the class `TestDeepEpDeepseekV32`
```python
    """Testcase: Verify that for the DeepSeek V3.2 model in the single-machine colocation scenario,
    its inference accuracy on the MMLU and GSM8K dataset meets the preset standard when the parameter --deepep-mode auto is configured.

    [Test Category] Expert Parallelism
    [Test Target] --moe-a2a-backend deepep;--deepep-mode
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDeepEpDeepseekV32`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDeepEpDeepseekV32`的设计意图。

### Lines 21-59: Declare TestDeepEpDeepseekV32 configuration
```python
    model = DEEPSEEK_V3_2_W8A8_WEIGHTS_PATH

    timeout_for_server_launch = 60000
    other_args = [
        "--trust-remote-code",
        "--tp-size",
        "16",
        "--quantization",
        "modelslim",
        "--moe-a2a-backend",
        "deepep",
        "--deepep-mode",
        "auto",
        "--mem-fraction-static",
        0.82,
        "--disable-cuda-graph",
        "--disable-radix-cache",
        "--context-length",
        40960,
        "--max-prefill-tokens",
        40960,
        "--max-total-tokens",
        40960,
    ]

    env = {
        **os.environ,
        "PYTORCH_NPU_ALLOC_CONF": "expandable_segments:True",
        "STREAMS_PER_DEVICE": "32",
        "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "16",
        "HCCL_BUFFSIZE": "1600",
        "HCCL_OP_EXPANSION_MODE": "AIV",
        "SGLANG_NPU_USE_MLAPO": "0",
        "SGLANG_NPU_USE_MULTI_STREAM": "1",
        "TASK_QUEUE_ENABLE": "0",
    }

    accuracy = 0.95  # Test GSM8K accuracy ≥0.95
    accuracy_mmlu = 0.85  # Test MMLU accuracy ≥0.85
```
**EN:** This block defines class-level settings that are shared across the `TestDeepEpDeepseekV32` test methods.
**CN:** 该代码块定义了 `TestDeepEpDeepseekV32` 各测试方法共享的类级配置。

### Lines 62-63: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ascend.gsm8k_ascend_mixin`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ascend.test_mmlu`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
