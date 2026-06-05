# test_npu_qwen3_30b.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/llm_models/test_npu_qwen3_30b.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on llm models npu qwen3 30b in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 llm models npu qwen3 30b 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies
```python
import unittest

from sglang.test.ascend.gsm8k_ascend_mixin import GSM8KAscendMixin
from sglang.test.ascend.test_ascend_utils import (
    QWEN3_30B_A3B_INSTRUCT_2507_WEIGHTS_PATH,
)
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 10-10: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-2-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 13-13: Define class TestQwen330B
```python
class TestQwen330B(GSM8KAscendMixin, CustomTestCase):
```
**EN:** This declaration introduces the `TestQwen330B` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen330B` 测试类，并说明它通过继承承担的职责。

### Lines 14-18: Document the class `TestQwen330B`
```python
    """Testcase: Verify that the inference accuracy of the Qwen/Qwen3-30B-A3B-Instruct-2507 model on the GSM8K dataset is no less than 0.90.

    [Test Category] Model
    [Test Target] Qwen/Qwen3-30B-A3B-Instruct-2507
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestQwen330B`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestQwen330B`的设计意图。

### Lines 20-35: Declare TestQwen330B configuration
```python
    model = QWEN3_30B_A3B_INSTRUCT_2507_WEIGHTS_PATH
    accuracy = 0.90
    other_args = [
        "--trust-remote-code",
        "--mem-fraction-static",
        0.7,
        "--max-running-requests",
        32,
        "--attention-backend",
        "ascend",
        "--disable-cuda-graph",
        "--cuda-graph-max-bs",
        32,
        "--tp-size",
        2,
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestQwen330B` test methods.
**CN:** 该代码块定义了 `TestQwen330B` 各测试方法共享的类级配置。

### Lines 38-39: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
