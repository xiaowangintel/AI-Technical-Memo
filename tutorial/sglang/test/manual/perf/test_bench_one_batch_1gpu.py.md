# test_bench_one_batch_1gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/perf/test_bench_one_batch_1gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `bench one batch 1gpu` scenario in `test/manual/perf`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/perf` 中的 `bench one batch 1gpu` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Process control logic / 进程控制逻辑
```python
import os
import re
import subprocess
import unittest

import numpy as np

from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    kill_process_tree,
    run_bench_one_batch,
    write_github_step_summary,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 19-20: Class definition for TestBenchOneBatch1GPU / 类定义
```python
class TestBenchOneBatch1GPU(CustomTestCase):
```
**EN:** This range declares `TestBenchOneBatch1GPU`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 21-25: Test routines around test_bs1_small / 测试例程
```python
    def test_bs1_small(self):
        _, output_throughput, _ = run_bench_one_batch(
            DEFAULT_SMALL_MODEL_NAME_FOR_TEST, ["--cuda-graph-max-bs", "2"]
        )
        self.assertGreater(output_throughput, 50)
```
**EN:** This range defines concrete test routine(s) `test_bs1_small`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `run_bench_one_batch` and `assertGreater`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 27-43: Test routines around test_bs1_default / 测试例程
```python
    def test_bs1_default(self):
        env = os.environ.copy()
        env["SGLANG_ENABLE_METRICS_DEVICE_TIMER"] = "1"

        command = [
            "python3",
            "-m",
            "sglang.bench_offline_throughput",
            "--num-prompts",
            "1",
            "--dataset-name",
            "random",
            "--random-input-len",
            "256",
            "--random-output-len",
            "1024",
            "--model-path",
```
**EN:** This range defines concrete test routine(s) `test_bs1_default`. The logic drives the target scenario and encodes the expected acceptance criteria. Environment variables are read here so the scenario adapts to the local machine and accelerator topology. Representative call sites include `copy`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里会读取环境变量，使场景能够适配本地机器与加速器拓扑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 44-51: Process control logic / 进程控制逻辑
```python
            DEFAULT_MODEL_NAME_FOR_TEST,
            "--cuda-graph-max-bs",
            "2",
        ]

        print(f"command={' '.join(command)}")
        process = subprocess.Popen(
            command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, env=env
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `join` and `Popen`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-68: Process control logic / 进程控制逻辑
```python
        )

        try:
            stdout, stderr = process.communicate()
            output = stdout.decode(errors="backslashreplace")
            error = stderr.decode(errors="backslashreplace")
            print(f"Output: {output}", flush=True)
            print(f"Error: {error}", flush=True)

            output_throughput = -1
            for line in output.split("\n"):
                if "Last generation throughput (tok/s):" in line:
                    output_throughput = float(line.split(":")[-1])
        finally:
            kill_process_tree(process.pid)

        if is_in_ci():
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `communicate`, `decode`, `split` and `throughput`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 69-76: Assertions and result checks / 断言与结果检查
```python
            write_github_step_summary(
                f"### test_bs1_default (llama-3.1-8b)\n"
                f"output_throughput: {output_throughput:.2f} token/s\n"
            )
            self.assertGreater(output_throughput, 135)

        fwd_occupancy_values = []
        for line in error.split("\n"):
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `write_github_step_summary`, `test_bs1_default`, `assertGreater` and `split`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 77-90: Assertions and result checks / 断言与结果检查
```python
            match = re.search(r"fwd occupancy:\s*([\d.]+|nan)%", line)
            if match:
                val = match.group(1)
                if val != "nan":
                    fwd_occupancy_values.append(float(val))

        print(f"{fwd_occupancy_values=}", flush=True)
        self.assertGreater(
            len(fwd_occupancy_values), 0, "No fwd occupancy values found in logs"
        )

        fwd_occupancy_p90 = float(np.percentile(fwd_occupancy_values, 90))
        print(f"{fwd_occupancy_p90=}", flush=True)
        self.assertGreater(fwd_occupancy_p90, 97.5)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `search`, `group`, `append` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 91-94: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- Environment-aware configuration / 环境感知配置
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `os`, `re`, `subprocess`, `unittest`
- **Third-party / 第三方库**: `numpy`
- **Project Modules / 项目模块**: `sglang.test.test_utils`
