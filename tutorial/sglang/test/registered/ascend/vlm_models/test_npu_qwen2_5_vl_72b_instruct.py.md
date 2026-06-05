# test_npu_qwen2_5_vl_72b_instruct.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/vlm_models/test_npu_qwen2_5_vl_72b_instruct.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on vlm models npu qwen2 5 vl 72b instruct in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 vlm models npu qwen2 5 vl 72b instruct 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Import dependencies
```python
import unittest

from sglang.test.ascend.test_ascend_utils import QWEN2_5_VL_72B_INSTRUCT_WEIGHTS_PATH
from sglang.test.ascend.vlm_utils import TestVLMModels
from sglang.test.ci.ci_register import register_npu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 7-7: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-8-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 10-10: Define class TestQwen25VL72B
```python
class TestQwen25VL72B(TestVLMModels):
```
**EN:** This declaration introduces the `TestQwen25VL72B` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen25VL72B` 测试类，并说明它通过继承承担的职责。

### Lines 11-15: Document the class `TestQwen25VL72B`
```python
    """Testcase: Verify that the inference accuracy of the Qwen/Qwen2.5-VL-72B-Instruct model on the MMMU dataset is no less than 0.2.

    [Test Category] Model
    [Test Target] Qwen/Qwen2.5-VL-72B-Instruct
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestQwen25VL72B`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestQwen25VL72B`的设计意图。

### Lines 17-33: Declare TestQwen25VL72B configuration
```python
    model = QWEN2_5_VL_72B_INSTRUCT_WEIGHTS_PATH
    mmmu_accuracy = 0.2
    other_args = [
        "--trust-remote-code",
        "--cuda-graph-max-bs",
        "32",
        "--enable-multimodal",
        "--mem-fraction-static",
        0.6,
        "--log-level",
        "info",
        "--attention-backend",
        "ascend",
        "--disable-cuda-graph",
        "--tp-size",
        8,
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestQwen25VL72B` test methods.
**CN:** 该代码块定义了 `TestQwen25VL72B` 各测试方法共享的类级配置。

### Lines 35-36: Run test: vlm mmmu benchmark
```python
    def test_vlm_mmmu_benchmark(self):
        self._run_vlm_mmmu_test()
```
**EN:** This test method exercises vlm mmmu benchmark and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 vlm mmmu benchmark 场景，并验证观测到的行为是否符合预期契约。

### Lines 39-40: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ascend.test_ascend_utils`, `sglang.test.ascend.vlm_utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `unittest.main`
