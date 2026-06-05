# test_npu_eagle3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/speculative_inference/test_npu_eagle3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on speculative inference npu eagle3 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 speculative inference npu eagle3 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Import dependencies
```python
import os
import unittest

from sglang.test.ascend.gsm8k_ascend_mixin import GSM8KAscendMixin
from sglang.test.ascend.test_ascend_utils import (
    QWEN3_8B_EAGLE3_WEIGHTS_PATH,
    QWEN3_8B_WEIGHTS_PATH,
)
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 12-12: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-1-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 15-15: Define class TestNpuEagle3
```python
class TestNpuEagle3(GSM8KAscendMixin, CustomTestCase):
```
**EN:** This declaration introduces the `TestNpuEagle3` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNpuEagle3` 测试类，并说明它通过继承承担的职责。

### Lines 16-20: Document the class `TestNpuEagle3`
```python
    """Testcase: Verify GSM8K inference accuracy ≥0.81 for model with specified EAGLE3 speculative inference parameters.

    [Test Category] Speculative Decoding
    [Test Target] --speculative-draft-model-quantization; --speculative-algorithm; --speculative-draft-model-path; --speculative-num-steps; --speculative-eagle-topk; --speculative-num-draft-tokens; --speculative-attention-mode
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNpuEagle3`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNpuEagle3`的设计意图。

### Lines 22-58: Declare TestNpuEagle3 configuration
```python
    model = QWEN3_8B_WEIGHTS_PATH
    timeout_for_server_launch = 1500
    other_args = [
        "--trust-remote-code",
        "--attention-backend",
        "ascend",
        "--disable-radix-cache",
        "--speculative-draft-model-quantization",
        "unquant",
        "--speculative-algorithm",
        "EAGLE3",
        "--speculative-draft-model-path",
        QWEN3_8B_EAGLE3_WEIGHTS_PATH,
        "--speculative-num-steps",
        "4",
        "--speculative-eagle-topk",
        "1",
        "--speculative-num-draft-tokens",
        "5",
        "--speculative-attention-mode",
        "decode",
        "--tp-size",
        "1",
        "--mem-fraction-static",
        "0.7",
        "--disable-cuda-graph",
        "--dtype",
        "bfloat16",
    ]

    env = {
        **os.environ,
        "SGLANG_ENABLE_OVERLAP_PLAN_STREAM": "1",
    }

    accuracy = 0.81
    num_questions = 1319
```
**EN:** This block defines class-level settings that are shared across the `TestNpuEagle3` test methods.
**CN:** 该代码块定义了 `TestNpuEagle3` 各测试方法共享的类级配置。

### Lines 61-62: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ascend.gsm8k_ascend_mixin`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
