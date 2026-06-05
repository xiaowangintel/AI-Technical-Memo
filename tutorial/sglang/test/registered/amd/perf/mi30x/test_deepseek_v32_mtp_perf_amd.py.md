# test_deepseek_v32_mtp_perf_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/perf/mi30x/test_deepseek_v32_mtp_perf_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on mi30x deepseek v32 mtp perf amd in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 mi30x deepseek v32 mtp perf amd 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Document the module
```python
"""AMD Nightly performance benchmark for DeepSeek-V3.2 model (MTP variant).

This test benchmarks the DeepSeek-V3.2 model with MTP (EAGLE speculative decoding)
configuration on 8 GPUs.

The model path can be configured via DEEPSEEK_V32_MODEL_PATH environment variable.

Registry: nightly-perf-8-gpu-deepseek-v32-mtp suite

Example usage:
    DEEPSEEK_V32_MODEL_PATH=deepseek-ai/DeepSeek-V3.2 python -m pytest test_deepseek_v32_mtp_perf_amd.py -v
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 14-21: Import dependencies
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

### Lines 24-26: Register CI metadata
```python
register_amd_ci(
    est_time=7200, suite="nightly-perf-8-gpu-deepseek-v32-mtp", nightly=True
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 29-57: Define helper: generate simple markdown report
```python
def generate_simple_markdown_report(results: List[BenchmarkResult]) -> str:
    """Generate a simplified markdown report without traces and cost columns.

    Skips the first result if it's a warmup run (duplicate batch_size).
    """
    model_header = results[0].model_path
    if results[0].run_name and results[0].run_name != "default":
        model_header += f" ({results[0].run_name})"

    gpu_config = os.getenv("GPU_CONFIG", "MI325")
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

### Lines 61-64: Define module constants
```python
DEEPSEEK_V32_MODEL_PATH = os.environ.get(
    "DEEPSEEK_V32_MODEL_PATH", "deepseek-ai/DeepSeek-V3.2"
)
PROFILE_DIR = "performance_profiles_deepseek_v32_mtp_mi325"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 67-67: Define class TestNightlyDeepseekV32MTPPerformance
```python
class TestNightlyDeepseekV32MTPPerformance(unittest.TestCase):
```
**EN:** This declaration introduces the `TestNightlyDeepseekV32MTPPerformance` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNightlyDeepseekV32MTPPerformance` 测试类，并说明它通过继承承担的职责。

### Lines 68-71: Document the class `TestNightlyDeepseekV32MTPPerformance`
```python
    """AMD Nightly performance benchmark for DeepSeek-V3.2 model (MTP variant).

    Tests the DeepSeek-V3.2 model with MTP (EAGLE speculative decoding) on MI325/MI300X.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNightlyDeepseekV32MTPPerformance`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNightlyDeepseekV32MTPPerformance`的设计意图。

### Lines 73-114: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEEPSEEK_V32_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.batch_sizes = [1, 8, 16, 64]
        cls.input_lens = tuple(_parse_int_list_env("NIGHTLY_INPUT_LENS", "4096"))
        cls.output_lens = tuple(_parse_int_list_env("NIGHTLY_OUTPUT_LENS", "512"))

        # MTP variant configuration for DeepSeek-V3.2
        # MI325 uses aiter attention backend + EAGLE speculative decoding
        cls.variant_config = {
            "name": "mtp",
            "other_args": [
                "--trust-remote-code",
                "--tp",
                "8",
                "--attention-backend",
                "aiter",
                "--chunked-prefill-size",
                "131072",
                "--speculative-algorithm",
                "EAGLE",
                "--speculative-num-steps",
                "3",
                "--speculative-eagle-topk",
                "1",
                "--speculative-num-draft-tokens",
                "4",
                "--mem-fraction-static",
                "0.7",
                "--model-loader-extra-config",
                '{"enable_multithread_load": true}',
                "--watchdog-timeout",
                "1200",
            ],
            "env_vars": {"SGLANG_USE_AITER": "1"},
        }

        cls.runner = NightlyBenchmarkRunner(PROFILE_DIR, cls.__name__, cls.base_url)
        cls.runner.setup_profile_directory()
        # Override full_report to remove traces help text
        cls.runner.full_report = f"## {cls.__name__}\n"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 116-146: Run test: bench one batch
```python
    def test_bench_one_batch(self):
        """Run benchmark for MTP variant."""
        try:
            result_tuple = self.runner.run_benchmark_for_model(
                model_path=self.model,
                batch_sizes=self.batch_sizes,
                input_lens=self.input_lens,
                output_lens=self.output_lens,
                other_args=self.variant_config["other_args"],
                variant=self.variant_config["name"],
                extra_bench_args=["--trust-remote-code"],
                enable_profile=False,  # Disable profiling for AMD tests
            )
            results = result_tuple[0]
            success = result_tuple[1]
            avg_spec_accept_length = result_tuple[2] if len(result_tuple) > 2 else None

            # Log speculative decoding accept length
            if avg_spec_accept_length is not None:
                print(f"  avg_spec_accept_length={avg_spec_accept_length:.2f}")

            # Use simplified report format without traces
            if results:
                self.runner.full_report += (
                    generate_simple_markdown_report(results) + "\n"
                )

            if not success:
                raise AssertionError(f"Benchmark failed for {self.model} (MTP variant)")
        finally:
            self.runner.write_final_report()
```
**EN:** This test method exercises bench one batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bench one batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 149-150: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.nightly_bench_utils`, `sglang.test.nightly_utils`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `typing`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `unittest.main`
