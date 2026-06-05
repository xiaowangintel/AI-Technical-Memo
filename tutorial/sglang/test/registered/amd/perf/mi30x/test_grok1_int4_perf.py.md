# test_grok1_int4_perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/perf/mi30x/test_grok1_int4_perf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on mi30x grok1 int4 perf in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 mi30x grok1 int4 perf 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Document the module
```python
"""Nightly performance benchmark for Grok-1 INT4 (W4A8KV8).

This test benchmarks Grok-1 (314B MOE) with INT4 weight quantization on 8 GPUs.

Model paths can be configured via environment variables:
- GROK1_MODEL_PATH: Path to Grok-1 INT4 model (default: amd/grok-1-W4A8KV8)
- GROK1_TOKENIZER_PATH: Path to Grok-1 tokenizer (default: Xenova/grok-1-tokenizer)

Example usage:
    python -m pytest test_grok1_int4_perf.py -v
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 13-20: Import dependencies
```python
import os
import unittest
from typing import List

from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.nightly_bench_utils import BenchmarkResult
from sglang.test.nightly_utils import NightlyBenchmarkRunner
from sglang.test.test_utils import DEFAULT_URL_FOR_TEST, _parse_int_list_env
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 23-23: Register CI metadata
```python
register_amd_ci(est_time=1500, suite="nightly-perf-8-gpu-grok1-int4", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 26-54: Define helper: generate simple markdown report
```python
def generate_simple_markdown_report(results: List[BenchmarkResult]) -> str:
    """Generate a simplified markdown report without traces and cost columns.

    Skips the first result if it's a warmup run (duplicate batch_size).
    """
    model_header = results[0].model_path
    if results[0].run_name and results[0].run_name != "default":
        model_header += f" ({results[0].run_name})"

    gpu_config = os.getenv("GPU_CONFIG", "")
    if gpu_config:
        model_header += f" [{gpu_config}]"

    summary = f"### {model_header}\n"
    summary += "| batch size | input len | latency (s) | input throughput (tok/s) | output throughput (tok/s) | ITL (ms) |\n"
    summary += "| ---------- | --------- | ----------- | ------------------------ | ------------------------- | -------- |\n"

    # Skip first result if it's a warmup (same batch_size as second result)
    report_results = (
        results[1:]
        if len(results) > 1 and results[0].batch_size == results[1].batch_size
        else results
    )

    for result in report_results:
        itl = 1 / (result.output_throughput / result.batch_size) * 1000
        summary += f"| {result.batch_size} | {result.input_len} | {result.latency:.2f} | {result.input_throughput:.2f} | {result.output_throughput:.2f} | {itl:.2f} |\n"

    return summary
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 58-60: Define module constants
```python
GROK1_MODEL_PATH = os.environ.get("GROK1_MODEL_PATH", "amd/grok-1-W4A8KV8")
GROK1_TOKENIZER_PATH = os.environ.get("GROK1_TOKENIZER_PATH", "Xenova/grok-1-tokenizer")
PROFILE_DIR = "performance_profiles_grok1_int4"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 63-63: Define class TestNightlyGrok1INT4Performance
```python
class TestNightlyGrok1INT4Performance(unittest.TestCase):
```
**EN:** This declaration introduces the `TestNightlyGrok1INT4Performance` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNightlyGrok1INT4Performance` 测试类，并说明它通过继承承担的职责。

### Lines 64-68: Document the class `TestNightlyGrok1INT4Performance`
```python
    """Nightly performance benchmark for Grok-1 INT4 (W4A8KV8).

    Tests Grok-1 (314B MOE) with INT4 weight quantization on TP=8.
    Runtime: ~25 minutes
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNightlyGrok1INT4Performance`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNightlyGrok1INT4Performance`的设计意图。

### Lines 70-102: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.batch_sizes = [1, 8, 16, 64]
        cls.input_lens = tuple(_parse_int_list_env("NIGHTLY_INPUT_LENS", "1024"))
        cls.output_lens = tuple(_parse_int_list_env("NIGHTLY_OUTPUT_LENS", "512"))

        cls.model_config = {
            "name": "grok1-int4",
            "model_path": GROK1_MODEL_PATH,
            "other_args": [
                "--trust-remote-code",
                "--tp",
                "8",
                "--quantization",
                "fp8",
                "--mem-fraction-static",
                "0.85",
                "--tokenizer-path",
                GROK1_TOKENIZER_PATH,
                "--attention-backend",
                "aiter",
            ],
            "env_vars": {
                "RCCL_MSCCL_ENABLE": "0",
                "SGLANG_USE_AITER": "1",
                "SGLANG_INT4_WEIGHT": "1",
            },
        }

        cls.runner = NightlyBenchmarkRunner(PROFILE_DIR, cls.__name__, cls.base_url)
        cls.runner.setup_profile_directory()
        cls.runner.full_report = f"## {cls.__name__}\n"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 104-140: Run test: bench grok1 int4
```python
    def test_bench_grok1_int4(self):
        """Run benchmark for Grok-1 INT4."""
        # Set environment variables
        old_env = {}
        for key, value in self.model_config.get("env_vars", {}).items():
            old_env[key] = os.environ.get(key)
            os.environ[key] = value
            print(f"Setting env: {key}={value}")

        try:
            result_tuple = self.runner.run_benchmark_for_model(
                model_path=self.model_config["model_path"],
                batch_sizes=self.batch_sizes,
                input_lens=self.input_lens,
                output_lens=self.output_lens,
                other_args=self.model_config["other_args"],
                variant=self.model_config["name"],
                extra_bench_args=["--trust-remote-code"],
                enable_profile=False,  # Disable profiling for AMD tests
            )
            results = result_tuple[0]
            success = result_tuple[1]

            if results:
                self.runner.full_report += (
                    generate_simple_markdown_report(results) + "\n"
                )

            self.assertTrue(success, "Benchmark failed for Grok-1 INT4")
        finally:
            # Restore original environment
            for key, value in old_env.items():
                if value is None:
                    os.environ.pop(key, None)
                else:
                    os.environ[key] = value
            self.runner.write_final_report()
```
**EN:** This test method exercises bench grok1 int4 and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bench grok1 int4 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 143-144: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.nightly_bench_utils`, `sglang.test.nightly_utils`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `typing`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `unittest.main`
