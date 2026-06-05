# test_npu_memory_consumption.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/test_npu_memory_consumption.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on ascend npu memory consumption in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 ascend npu memory consumption 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Document the module
```python
"""
Usage:
python3 -m unittest test_ascend_memory_consumption.TestMemoryConsumptionAscend.test_memory_consumption
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 6-18: Import dependencies
```python
import os
import unittest

import torch

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 20-24: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-2-npu-a3",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 26-27: Implement control flow
```python
if "ASCEND_RT_VISIBLE_DEVICES" not in os.environ:
    os.environ["ASCEND_RT_VISIBLE_DEVICES"] = "0,1"
```
**EN:** This block adds supporting control flow around the main test or helper logic.
**CN:** 该代码块围绕主要测试或辅助逻辑补充了控制流。

### Lines 28-31: Define module constants
```python
DEFAULT_PORT_FOR_SRT_TEST_RUNNER = (
    8000 + int(os.environ.get("ASCEND_RT_VISIBLE_DEVICES", "0")[0]) * 100
)
DEFAULT_URL_FOR_TEST = f"http://127.0.0.1:{DEFAULT_PORT_FOR_SRT_TEST_RUNNER + 1000}"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 34-34: Define class TestMemoryConsumptionAscend
```python
class TestMemoryConsumptionAscend(CustomTestCase):
```
**EN:** This declaration introduces the `TestMemoryConsumptionAscend` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMemoryConsumptionAscend` 测试类，并说明它通过继承承担的职责。

### Lines 36-76: Run test: memory consumption
```python
    def test_memory_consumption(self):

        model = "/root/.cache/modelscope/hub/models/Qwen/Qwen3-30B-A3B-w8a8"
        base_url = DEFAULT_URL_FOR_TEST

        ### Calculate initial used memory
        free_npu_memory, total_npu_memory = torch.npu.mem_get_info()
        initial_used_memory = total_npu_memory - free_npu_memory

        process = popen_launch_server(
            model,
            base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--trust-remote-code",
                "--device",
                "npu",
                "--attention-backend",
                "ascend",
                "--tp-size",
                "2",
                "--mem-fraction-static",
                "0.8",
                "--cuda-graph-bs",
                "1",
                "--max-total-tokens",
                "1024",
                "--disable-radix-cache",
                "--disable-cuda-graph",
            ],
        )

        ### Calculate initial used memory
        free_npu_memory, total_npu_memory = torch.npu.mem_get_info()
        used_memory_after_server_starting = (
            total_npu_memory - free_npu_memory - initial_used_memory
        ) / (1 << 30)
        self.assertLessEqual(float(used_memory_after_server_starting), 17.00)

        # Clean up everything
        kill_process_tree(process.pid)
```
**EN:** This test method exercises memory consumption and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 memory consumption 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 79-80: Expose unittest entrypoint
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
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `torch`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
