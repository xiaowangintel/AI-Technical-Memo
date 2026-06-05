# test_ascend_glm_4_5v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/vlm_models/test_ascend_glm_4_5v.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on vlm models ascend glm 4 5v in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 vlm models ascend glm 4 5v 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies
```python
import unittest

from sglang.test.ascend.vlm_utils import TestVLMModels
from sglang.test.ci.ci_register import register_npu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 6-6: Register CI metadata
```python
register_npu_ci(est_time=400, suite="nightly-8-npu-a3", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 9-9: Define class TestGLM4Models
```python
class TestGLM4Models(TestVLMModels):
```
**EN:** This declaration introduces the `TestGLM4Models` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGLM4Models` 测试类，并说明它通过继承承担的职责。

### Lines 10-26: Declare TestGLM4Models configuration
```python
    model = "/root/.cache/modelscope/hub/models/ZhipuAI/GLM-4.5V"
    mmmu_accuracy = 0.2
    other_args = [
        "--trust-remote-code",
        "--cuda-graph-max-bs",
        "32",
        "--enable-multimodal",
        "--mem-fraction-static",
        0.7,
        "--log-level",
        "info",
        "--attention-backend",
        "ascend",
        "--disable-cuda-graph",
        "--tp-size",
        8,
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestGLM4Models` test methods.
**CN:** 该代码块定义了 `TestGLM4Models` 各测试方法共享的类级配置。

### Lines 28-29: Run test: vlm mmmu benchmark
```python
    def test_vlm_mmmu_benchmark(self):
        self._run_vlm_mmmu_test()
```
**EN:** This test method exercises vlm mmmu benchmark and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 vlm mmmu benchmark 场景，并验证观测到的行为是否符合预期契约。

### Lines 32-33: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ascend.vlm_utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `unittest.main`
