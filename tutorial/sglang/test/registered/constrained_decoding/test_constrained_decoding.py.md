# test_constrained_decoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/constrained_decoding/test_constrained_decoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on constrained decoding in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 constrained decoding 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import unittest

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.ebnf_constrained_kit import EBNFConstrainedMixin
from sglang.test.kits.json_constrained_kit import JSONConstrainedMixin
from sglang.test.kits.regex_constrained_kit import RegexConstrainedMixin
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 16-17: Register CI metadata
```python
register_cuda_ci(est_time=120, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=179, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 20-20: Define class ServerWithGrammar
```python
class ServerWithGrammar(CustomTestCase):
```
**EN:** This declaration introduces the `ServerWithGrammar` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `ServerWithGrammar` 测试类，并说明它通过继承承担的职责。

### Lines 21-22: Declare ServerWithGrammar configuration
```python
    backend = "xgrammar"
    disable_overlap = False
```
**EN:** This block defines class-level settings that are shared across the `ServerWithGrammar` test methods.
**CN:** 该代码块定义了 `ServerWithGrammar` 各测试方法共享的类级配置。

### Lines 24-43: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        launch_args = [
            "--max-running-requests",
            "10",
            "--grammar-backend",
            cls.backend,
        ]

        if cls.disable_overlap:
            launch_args += ["--disable-overlap-schedule"]

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=launch_args,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 45-47: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 50-50: Define class TestXGrammarBackend
```python
class TestXGrammarBackend(
```
**EN:** This declaration introduces the `TestXGrammarBackend` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestXGrammarBackend` 测试类，并说明它通过继承承担的职责。

### Lines 56-56: Declare TestXGrammarBackend configuration
```python
    backend = "xgrammar"
```
**EN:** This block defines class-level settings that are shared across the `TestXGrammarBackend` test methods.
**CN:** 该代码块定义了 `TestXGrammarBackend` 各测试方法共享的类级配置。

### Lines 59-59: Define class TestOutlinesBackend
```python
class TestOutlinesBackend(ServerWithGrammar, JSONConstrainedMixin):
```
**EN:** This declaration introduces the `TestOutlinesBackend` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOutlinesBackend` 测试类，并说明它通过继承承担的职责。

### Lines 60-60: Declare TestOutlinesBackend configuration
```python
    backend = "outlines"
```
**EN:** This block defines class-level settings that are shared across the `TestOutlinesBackend` test methods.
**CN:** 该代码块定义了 `TestOutlinesBackend` 各测试方法共享的类级配置。

### Lines 63-63: Define class TestLLGuidanceBackend
```python
class TestLLGuidanceBackend(
```
**EN:** This declaration introduces the `TestLLGuidanceBackend` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLLGuidanceBackend` 测试类，并说明它通过继承承担的职责。

### Lines 69-69: Declare TestLLGuidanceBackend configuration
```python
    backend = "llguidance"
```
**EN:** This block defines class-level settings that are shared across the `TestLLGuidanceBackend` test methods.
**CN:** 该代码块定义了 `TestLLGuidanceBackend` 各测试方法共享的类级配置。

### Lines 72-73: Expose unittest entrypoint
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
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.ebnf_constrained_kit`, `sglang.test.kits.json_constrained_kit`, `sglang.test.kits.regex_constrained_kit`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
