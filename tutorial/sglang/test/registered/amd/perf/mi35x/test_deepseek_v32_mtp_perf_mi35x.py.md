# test_deepseek_v32_mtp_perf_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/perf/mi35x/test_deepseek_v32_mtp_perf_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on deepseek v32 mtp perf mi35x in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 deepseek v32 mtp perf mi35x 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Document the module
```python
"""MI35x Nightly performance benchmark for DeepSeek-V3.2 model (MTP variant).

This test benchmarks the DeepSeek-V3.2 model with MTP (EAGLE speculative decoding)
configuration on 8 GPUs.

The model path can be configured via DEEPSEEK_V32_MODEL_PATH environment variable.

Registry: nightly-perf-8-gpu-mi35x-deepseek-v32-mtp suite

Example usage:
    DEEPSEEK_V32_MODEL_PATH=deepseek-ai/DeepSeek-V3.2 python -m pytest test_deepseek_v32_mtp_perf_mi35x.py -v
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 14-26: Import dependencies
```python
import os
import unittest
from typing import List, Optional, Tuple

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.nightly_bench_utils import BenchmarkResult
from sglang.test.nightly_utils import NightlyBenchmarkRunner
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    _parse_int_list_env,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 29-31: Register CI metadata
```python
register_amd_ci(
    est_time=5400, suite="nightly-perf-8-gpu-mi35x-deepseek-v32-mtp", nightly=True
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 34-62: Define helper: generate simple markdown report
```python
def generate_simple_markdown_report(results: List[BenchmarkResult]) -> str:
    """Generate a simplified markdown report without traces and cost columns.

    Skips the first result if it's a warmup run (duplicate batch_size).
    """
    model_header = results[0].model_path
    if results[0].run_name and results[0].run_name != "default":
        model_header += f" ({results[0].run_name})"

    gpu_config = os.getenv("GPU_CONFIG", "MI35x")
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

### Lines 65-109: Define helper: run benchmark with timeout
```python
def _run_benchmark_with_timeout(
    runner: NightlyBenchmarkRunner,
    model_path: str,
    batch_sizes: List[int],
    input_lens: Tuple[int, ...],
    output_lens: Tuple[int, ...],
    other_args: List[str],
    variant: str,
    extra_bench_args: Optional[List[str]],
    timeout: int,
) -> Tuple[List[BenchmarkResult], bool, Optional[float]]:
    """Run benchmark with a custom server launch timeout."""
    model_description = f"{model_path}" + (f" ({variant})" if variant else "")
    process = popen_launch_server(
        model=model_path,
        base_url=runner.base_url,
        other_args=other_args,
        timeout=timeout,
    )
    try:
        profile_path_prefix, json_output_file = runner.generate_profile_filename(
            model_path, variant
        )
        bench_args = list(extra_bench_args) if extra_bench_args else []
        if variant:
            bench_args.extend(["--run-name", variant])
        command = runner.build_benchmark_command(
            model_path,
            batch_sizes,
            input_lens,
            output_lens,
            profile_path_prefix,
            json_output_file,
            extra_args=bench_args,
            enable_profile=False,  # Disable profiling for AMD tests
        )
        _, cmd_success = runner.run_benchmark_command(command, model_description)
        if not cmd_success:
            return [], False, None
        benchmark_results, load_success = runner.load_benchmark_results(
            json_output_file, model_description
        )
        return benchmark_results, load_success, None
    finally:
        kill_process_tree(process.pid)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 113-117: Define module constants
```python
DEEPSEEK_V32_MODEL_PATH = os.environ.get(
    "DEEPSEEK_V32_MODEL_PATH", "deepseek-ai/DeepSeek-V3.2"
)
PROFILE_DIR = "performance_profiles_deepseek_v32_mtp"
SERVER_LAUNCH_TIMEOUT = 5400
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 120-120: Define class TestNightlyDeepseekV32MTPPerformance
```python
class TestNightlyDeepseekV32MTPPerformance(unittest.TestCase):
```
**EN:** This declaration introduces the `TestNightlyDeepseekV32MTPPerformance` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNightlyDeepseekV32MTPPerformance` 测试类，并说明它通过继承承担的职责。

### Lines 121-124: Document the class `TestNightlyDeepseekV32MTPPerformance`
```python
    """MI35x Nightly performance benchmark for DeepSeek-V3.2 model (MTP variant).

    Tests the DeepSeek-V3.2 model with MTP (EAGLE speculative decoding) on TP=8.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNightlyDeepseekV32MTPPerformance`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNightlyDeepseekV32MTPPerformance`的设计意图。

### Lines 126-166: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEEPSEEK_V32_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.batch_sizes = [1, 8, 16, 64]
        cls.input_lens = tuple(_parse_int_list_env("NIGHTLY_INPUT_LENS", "4096"))
        cls.output_lens = tuple(_parse_int_list_env("NIGHTLY_OUTPUT_LENS", "512"))

        # MTP variant configuration for DeepSeek-V3.2
        # MI35x uses tilelang NSA backends + EAGLE speculative decoding
        cls.variant_config = {
            "name": "mtp",
            "other_args": [
                "--trust-remote-code",
                "--tp",
                "8",
                "--nsa-prefill-backend",
                "tilelang",
                "--nsa-decode-backend",
                "tilelang",
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
        }

        cls.runner = NightlyBenchmarkRunner(PROFILE_DIR, cls.__name__, cls.base_url)
        cls.runner.setup_profile_directory()
        # Override full_report to remove traces help text
        cls.runner.full_report = f"## {cls.__name__}\n"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 168-199: Run test: bench one batch
```python
    def test_bench_one_batch(self):
        """Run benchmark for MTP variant."""
        try:
            result_tuple = _run_benchmark_with_timeout(
                runner=self.runner,
                model_path=self.model,
                batch_sizes=self.batch_sizes,
                input_lens=self.input_lens,
                output_lens=self.output_lens,
                other_args=self.variant_config["other_args"],
                variant=self.variant_config["name"],
                extra_bench_args=["--trust-remote-code"],
                timeout=SERVER_LAUNCH_TIMEOUT,
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

### Lines 202-203: Expose unittest entrypoint
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

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.nightly_bench_utils`, `sglang.test.nightly_utils`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `typing`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `unittest.main`
