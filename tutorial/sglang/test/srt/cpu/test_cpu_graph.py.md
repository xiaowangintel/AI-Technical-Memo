# test_cpu_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_cpu_graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates cpu graph behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 cpu graph 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: supporting statements / 辅助语句
```python
"""
Usage:
python3 -m unittest test_cpu_graph.TestCPUGraph.test_mmlu_torch_compile_cpu
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 6-21: module imports and dependencies / 模块导入与依赖
```python
import copy
import os
import unittest
from types import SimpleNamespace

from sglang.srt.utils import get_cpu_ids_by_node, kill_process_tree
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
**EN:** This block imports the modules needed by the rest of the file, including `copy`, `os`, `unittest`, `types`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `copy`, `os`, `unittest`, `types`。

### Lines 24-25: class TestCPUGraph declaration / 类 TestCPUGraph 声明
```python
class TestCPUGraph(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 26-41: test case latency torch compile cpu / 测试用例 latency torch compile cpu
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
**EN:** This test exercises `test_latency_torch_compile_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_latency_torch_compile_cpu`。

### Lines 43-84: test case mmlu torch compile cpu / 测试用例 mmlu torch compile cpu
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
**EN:** This test exercises `test_mmlu_torch_compile_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu_torch_compile_cpu`。

### Lines 87-88: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestCPUGraph`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCPUGraph.test_latency_torch_compile_cpu`: This test exercises `test_latency_torch_compile_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_latency_torch_compile_cpu`。
- `TestCPUGraph.test_mmlu_torch_compile_cpu`: This test exercises `test_mmlu_torch_compile_cpu` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu_torch_compile_cpu`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `os`, `unittest`, `types`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 88
