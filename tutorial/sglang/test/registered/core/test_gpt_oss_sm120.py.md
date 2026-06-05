# test_gpt_oss_sm120.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_gpt_oss_sm120.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core gpt oss sm120 in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 core gpt oss sm120 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Import dependencies
```python
import unittest

import torch

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.gpt_oss_common import BaseTestGptOss
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 8-8: Register CI metadata
```python
register_cuda_ci(est_time=345, stage="extra-a", runner_config="1-gpu-small")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 11-12: Define class TestGptOssSm120
```python
@unittest.skipIf(not torch.cuda.is_available(), "CUDA is not available")
class TestGptOssSm120(BaseTestGptOss):
```
**EN:** This declaration introduces the `TestGptOssSm120` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGptOssSm120` 测试类，并说明它通过继承承担的职责。

### Lines 13-19: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        compute_capability = torch.cuda.get_device_capability()
        if compute_capability != (12, 0):
            raise unittest.SkipTest(
                f"GPT-OSS SM120 test requires SM 12.0, but found {compute_capability[0]}.{compute_capability[1]}"
            )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 21-30: Run test: mxfp4 20b
```python
    def test_mxfp4_20b(self):
        self.run_test(
            model_variant="20b",
            quantization="mxfp4",
            expected_score_of_reasoning_effort={
                "low": 0.34,
                "medium": 0.34,
                "high": 0.27,
            },
        )
```
**EN:** This test method exercises mxfp4 20b and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 mxfp4 20b 场景，并验证观测到的行为是否符合预期契约。

### Lines 33-34: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.gpt_oss_common`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`
