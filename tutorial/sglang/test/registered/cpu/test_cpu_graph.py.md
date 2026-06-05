# test_cpu_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_cpu_graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu graph in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 cpu graph 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Document the module
```python
"""
Usage:
python3 -m unittest test_cpu_graph.TestCPUGraph.test_mmlu_torch_compile_cpu
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 6-22: Import dependencies
```python
import copy
import os
import unittest
from types import SimpleNamespace

from sglang.srt.utils import get_cpu_ids_by_node, kill_process_tree
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    intel_amx_benchmark,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 24-24: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 27-27: Define class TestCPUGraph
```python
class TestCPUGraph(CustomTestCase):
```
**EN:** This declaration introduces the `TestCPUGraph` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCPUGraph` 测试类，并说明它通过继承承担的职责。

### Lines 29-44: Run test: latency torch compile cpu
```python
    @intel_amx_benchmark(
        extra_args=[
            "--batch-size",
            "1",
            "--mem-fraction-static",
            "0.05",
            "--enable-torch-compile",
            "--torch-compile-max-bs",
            "2",
            "--cuda-graph-bs",
            "2",
        ],
        min_throughput=7,
    )
    def test_latency_torch_compile_cpu(self):
        return DEFAULT_MLA_MODEL_NAME_FOR_TEST
```
**EN:** This test method exercises latency torch compile cpu and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 latency torch compile cpu 场景，并验证观测到的行为是否符合预期契约。

### Lines 46-87: Run test: mmlu torch compile cpu
```python
    def test_mmlu_torch_compile_cpu(self):
        model = DEFAULT_MLA_MODEL_NAME_FOR_TEST
        base_url = DEFAULT_URL_FOR_TEST
        cpu_ids_by_node = get_cpu_ids_by_node()
        n_numa_node = len(cpu_ids_by_node)
        env = copy.deepcopy(os.environ)
        env["SGLANG_CPU_OMP_THREADS_BIND"] = "all"
        process = popen_launch_server(
            model,
            base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--attention-backend",
                "intel_amx",
                "--mem-fraction-static",
                "0.05",
                "--disable-radix",
                "--trust-remote-code",
                "--disable-overlap-schedule",
                "--enable-torch-compile",
                "--cuda-graph-bs",
                "2",
                "--tp",
                f"{n_numa_node}",
            ],
            env=env,
        )

        try:
            args = SimpleNamespace(
                base_url=base_url,
                model=model,
                eval_name="mmlu",
                num_examples=64,
                num_threads=32,
            )

            metrics = run_eval(args)
            if is_in_ci():
                self.assertGreater(metrics["score"], 0.45)
        finally:
            kill_process_tree(process.pid)
```
**EN:** This test method exercises mmlu torch compile cpu and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 mmlu torch compile cpu 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 90-91: Expose unittest entrypoint
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
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `copy`, `os`, `types`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
