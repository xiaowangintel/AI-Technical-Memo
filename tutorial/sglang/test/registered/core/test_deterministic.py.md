# test_deterministic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_deterministic.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core deterministic in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 core deterministic 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Document the module
```python
"""
Usage:
cd test/srt
python3 -m unittest test_deterministic.TestDeterministic.TESTCASE

Note that there is also `python/sglang/test/test_deterministic.py` as an interactive test. We are converting that
test into unit tests so that's easily reproducible in CI.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 10-17: Import dependencies
```python
import unittest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_deterministic_utils import (
    COMMON_SERVER_ARGS,
    TestDeterministicBase,
)
from sglang.test.test_utils import is_in_amd_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 19-20: Register CI metadata
```python
register_cuda_ci(est_time=207, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=278, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 23-24: Define class TestFlashinferDeterministic
```python
@unittest.skipIf(is_in_amd_ci(), "Skip for AMD CI.")
class TestFlashinferDeterministic(TestDeterministicBase):
```
**EN:** This declaration introduces the `TestFlashinferDeterministic` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFlashinferDeterministic` 测试类，并说明它通过继承承担的职责。

### Lines 26-35: Define helper: get server args
```python
    @classmethod
    def get_server_args(cls):
        args = COMMON_SERVER_ARGS
        args.extend(
            [
                "--attention-backend",
                "flashinfer",
            ]
        )
        return args
```
**EN:** This helper function encapsulates reusable logic inside `TestFlashinferDeterministic` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFlashinferDeterministic` 内部调用，从而让场景结构更清晰。

### Lines 38-39: Define class TestFa3Deterministic
```python
@unittest.skipIf(is_in_amd_ci(), "Skip for AMD CI.")
class TestFa3Deterministic(TestDeterministicBase):
```
**EN:** This declaration introduces the `TestFa3Deterministic` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFa3Deterministic` 测试类，并说明它通过继承承担的职责。

### Lines 41-50: Define helper: get server args
```python
    @classmethod
    def get_server_args(cls):
        args = COMMON_SERVER_ARGS
        args.extend(
            [
                "--attention-backend",
                "fa3",
            ]
        )
        return args
```
**EN:** This helper function encapsulates reusable logic inside `TestFa3Deterministic` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFa3Deterministic` 内部调用，从而让场景结构更清晰。

### Lines 53-53: Define class TestTritonDeterministic
```python
class TestTritonDeterministic(TestDeterministicBase):
```
**EN:** This declaration introduces the `TestTritonDeterministic` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTritonDeterministic` 测试类，并说明它通过继承承担的职责。

### Lines 55-64: Define helper: get server args
```python
    @classmethod
    def get_server_args(cls):
        args = COMMON_SERVER_ARGS
        args.extend(
            [
                "--attention-backend",
                "triton",
            ]
        )
        return args
```
**EN:** This helper function encapsulates reusable logic inside `TestTritonDeterministic` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestTritonDeterministic` 内部调用，从而让场景结构更清晰。

### Lines 67-68: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.test_deterministic_utils`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `unittest.main`
