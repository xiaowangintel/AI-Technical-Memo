# test_vlms_perf_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/perf/mi30x/test_vlms_perf_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on mi30x vlms perf amd in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 mi30x vlms perf amd 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Document the module
```python
"""AMD Nightly performance benchmark for VLM models (2-GPU).

This test benchmarks Vision-Language Models on AMD MI30x/MI35x with 2 GPUs.

Registry: nightly-amd-perf-vlm-2-gpu suite

Example usage:
    python -m pytest test_vlms_perf_amd.py -v
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 11-24: Import dependencies
```python
import os
import unittest
import warnings
from typing import List

from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.nightly_bench_utils import BenchmarkResult
from sglang.test.nightly_utils import NightlyBenchmarkRunner
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    ModelLaunchSettings,
    _parse_int_list_env,
    parse_models,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 27-27: Register CI metadata
```python
register_amd_ci(est_time=7200, suite="nightly-amd-perf-vlm-2-gpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 29-29: Define module constants
```python
PROFILE_DIR = "performance_profiles_vlms_amd"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 32-41: Define module constants
```python
MODEL_DEFAULTS = [
    ModelLaunchSettings(
        "Qwen/Qwen2.5-VL-7B-Instruct",
        extra_args=["--mem-fraction-static=0.7"],
    ),
    ModelLaunchSettings(
        "Qwen/Qwen3-VL-30B-A3B-Instruct",
        tp_size=2,
    ),
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 44-72: Define helper: generate simple markdown report
```python
def generate_simple_markdown_report(results: List[BenchmarkResult]) -> str:
    """Generate a simplified markdown report without traces and cost columns.

    Skips the first result if it's a warmup run (duplicate batch_size).
    """
    model_header = results[0].model_path
    if results[0].run_name and results[0].run_name != "default":
        model_header += f" ({results[0].run_name})"

    gpu_config = os.getenv("GPU_CONFIG", "AMD")
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

### Lines 75-75: Define class TestNightlyVLMsPerfAMD
```python
class TestNightlyVLMsPerfAMD(unittest.TestCase):
```
**EN:** This declaration introduces the `TestNightlyVLMsPerfAMD` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNightlyVLMsPerfAMD` 测试类，并说明它通过继承承担的职责。

### Lines 76-76: Document the class `TestNightlyVLMsPerfAMD`
```python
    """AMD Nightly performance benchmark for VLM models (2-GPU)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNightlyVLMsPerfAMD`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNightlyVLMsPerfAMD`的设计意图。

### Lines 78-100: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        warnings.filterwarnings(
            "ignore", category=ResourceWarning, message="unclosed.*socket"
        )

        nightly_vlm_models_str = os.environ.get("NIGHTLY_VLM_MODELS")
        if nightly_vlm_models_str:
            cls.models = []
            model_paths = parse_models(nightly_vlm_models_str)
            for model_path in model_paths:
                cls.models.append(ModelLaunchSettings(model_path))
        else:
            cls.models = MODEL_DEFAULTS

        cls.base_url = DEFAULT_URL_FOR_TEST
        # First batch_size=1 is warmup (standalone job, no accuracy test to warm up)
        cls.batch_sizes = _parse_int_list_env("NIGHTLY_VLM_BATCH_SIZES", "1,1,2,8,16")
        cls.input_lens = tuple(_parse_int_list_env("NIGHTLY_VLM_INPUT_LENS", "4096"))
        cls.output_lens = tuple(_parse_int_list_env("NIGHTLY_VLM_OUTPUT_LENS", "512"))
        cls.runner = NightlyBenchmarkRunner(PROFILE_DIR, cls.__name__, cls.base_url)
        cls.runner.setup_profile_directory()
        cls.runner.full_report = f"## {cls.__name__}\n"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 102-142: Run test: bench one batch
```python
    def test_bench_one_batch(self):
        """Run benchmark for all configured VLM models."""
        all_model_succeed = True

        try:
            for model_setup in self.models:
                with self.subTest(model=model_setup.model_path):
                    other_args = list(model_setup.extra_args or [])
                    if model_setup.tp_size and model_setup.tp_size > 1:
                        other_args.extend(["--tp", str(model_setup.tp_size)])

                    # VLMs need additional benchmark args for dataset and trust-remote-code
                    extra_bench_args = [
                        "--trust-remote-code",
                        "--dataset-name=mmmu",
                    ]

                    result_tuple = self.runner.run_benchmark_for_model(
                        model_path=model_setup.model_path,
                        batch_sizes=self.batch_sizes,
                        input_lens=self.input_lens,
                        output_lens=self.output_lens,
                        other_args=other_args,
                        extra_bench_args=extra_bench_args,
                        enable_profile=False,  # Disable profiling for AMD tests
                    )
                    results = result_tuple[0]
                    success = result_tuple[1]

                    if not success:
                        all_model_succeed = False

                    if results:
                        self.runner.full_report += (
                            generate_simple_markdown_report(results) + "\n"
                        )
        finally:
            self.runner.write_final_report()

        if not all_model_succeed:
            raise AssertionError("Some models failed the perf tests.")
```
**EN:** This test method exercises bench one batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bench one batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 145-146: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `os`, `typing`, `unittest`, `warnings`
- Notable symbols / 关键符号: `register_amd_ci`, `unittest.main`
