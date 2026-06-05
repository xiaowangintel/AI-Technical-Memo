# test_npu_hierarchical_cache_mutually_exclusive.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/HiCache/test_npu_hierarchical_cache_mutually_exclusive.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on Hi Cache npu hierarchical cache mutually exclusive in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 Hi Cache npu hierarchical cache mutually exclusive 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Import dependencies
```python
import os
import unittest

from sglang.test.ascend.test_ascend_utils import QWEN3_8B_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务。

### Lines 13-17: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-1-npu-a3",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 20-20: Define class TestNpuHierarchicalCacheMutuallyExclusive
```python
class TestNpuHierarchicalCacheMutuallyExclusive(CustomTestCase):
```
**EN:** This declaration introduces the `TestNpuHierarchicalCacheMutuallyExclusive` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNpuHierarchicalCacheMutuallyExclusive` 测试类，并说明它通过继承承担的职责。

### Lines 21-26: Document the class `TestNpuHierarchicalCacheMutuallyExclusive`
```python
    """Testcase: The test parameter disable-radix-cache and enable-hierarchical-cache
                are mutually exclusive and cannot be used simultaneously.

    [Test Category] HiCache
    [Test Target] --disable-radix-cache; --enable-hierarchical-cache
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNpuHierarchicalCacheMutuallyExclusive`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNpuHierarchicalCacheMutuallyExclusive`的设计意图。

### Lines 28-64: Run test: hierarchical cache mutually exclusive
```python
    def test_hierarchical_cache_mutually_exclusive(self):
        error_message = (
            "The arguments enable-hierarchical-cache and disable-radix-cache are mutually exclusive and "
            "cannot be used at the same time. Please use only one of them."
        )
        other_args = [
            "--attention-backend",
            "ascend",
            "--disable-cuda-graph",
            "--mem-fraction-static",
            0.8,
            "--tp-size",
            2,
            "--enable-hierarchical-cache",
            "--disable-radix-cache",
        ]
        out_log_file = open("./cache_out_log.txt", "w+", encoding="utf-8")
        err_log_file = open("./cache_err_log.txt", "w+", encoding="utf-8")
        try:
            popen_launch_server(
                QWEN3_8B_WEIGHTS_PATH,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=other_args,
                return_stdout_stderr=(out_log_file, err_log_file),
            )
        except Exception as e:
            print(f"Server launch failed as expects:{e}")
        finally:
            err_log_file.seek(0)
            content = err_log_file.read()
            # error_message information is recorded in the error log
            self.assertIn(error_message, content)
            out_log_file.close()
            err_log_file.close()
            os.remove("./cache_out_log.txt")
            os.remove("./cache_err_log.txt")
```
**EN:** This test method exercises hierarchical cache mutually exclusive and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 hierarchical cache mutually exclusive 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并通过断言检查明确的预期。

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
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `CustomTestCase`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
