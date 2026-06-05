# test_deepseek_r1_mxfp4_kv_fp8_perf_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/perf/mi35x/test_deepseek_r1_mxfp4_kv_fp8_perf_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on deepseek r1 mxfp4 kv fp8 perf mi35x in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 deepseek r1 mxfp4 kv fp8 perf mi35x 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Document the module
```python
"""MI35x Nightly performance benchmark for DeepSeek-R1-MXFP4 model with KV Cache FP8.

This test benchmarks the DeepSeek-R1-MXFP4 quantized model on MI35x with 8 GPUs
using --kv-cache-dtype fp8_e4m3.

The model path can be configured via DEEPSEEK_R1_MXFP4_MODEL_PATH environment variable.

Registry: nightly-perf-8-gpu-mi35x-deepseek-r1-mxfp4-kv-fp8 suite

Example usage:
    DEEPSEEK_R1_MXFP4_MODEL_PATH=/data2/models/amd-DeepSeek-R1-MXFP4-Preview python -m pytest test_deepseek_r1_mxfp4_kv_fp8_perf_mi35x.py -v
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 14-14: Import dependencies
```python
import os
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 17-18: Implement expr logic
```python
os.environ.setdefault("HF_HOME", "/data2/models/huggingface")
os.environ.setdefault("HF_HUB_CACHE", "/data2/models/huggingface/hub")
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 19-25: Import dependencies
```python
import unittest
from typing import List

from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.nightly_bench_utils import BenchmarkResult
from sglang.test.nightly_utils import NightlyBenchmarkRunner
from sglang.test.test_utils import DEFAULT_URL_FOR_TEST, _parse_int_list_env
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 28-32: Register CI metadata
```python
register_amd_ci(
    est_time=18000,
    suite="nightly-perf-8-gpu-mi35x-deepseek-r1-mxfp4-kv-fp8",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 35-63: Define helper: generate simple markdown report
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

### Lines 68-70: Define module constants
```python
DEEPSEEK_R1_MXFP4_LOCAL_PATH = "/data2/models/amd-DeepSeek-R1-MXFP4-Preview"
DEEPSEEK_R1_MXFP4_HF_MODEL_ID = "amd/DeepSeek-R1-MXFP4-Preview"
PROFILE_DIR = "performance_profiles_deepseek_r1_mxfp4_kv_fp8_mi35x"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 73-83: Define helper: get model path
```python
def get_model_path() -> str:
    """Get effective model path: env var > local path > HF model ID."""
    # Check env var first
    env_path = os.environ.get("DEEPSEEK_R1_MXFP4_MODEL_PATH")
    if env_path:
        return env_path
    # Check local path
    if os.path.exists(DEEPSEEK_R1_MXFP4_LOCAL_PATH):
        return DEEPSEEK_R1_MXFP4_LOCAL_PATH
    # Fall back to HF model ID
    return DEEPSEEK_R1_MXFP4_HF_MODEL_ID
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 86-86: Define class TestDeepseekR1MXFP4KvFp8PerfMI35x
```python
class TestDeepseekR1MXFP4KvFp8PerfMI35x(unittest.TestCase):
```
**EN:** This declaration introduces the `TestDeepseekR1MXFP4KvFp8PerfMI35x` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepseekR1MXFP4KvFp8PerfMI35x` 测试类，并说明它通过继承承担的职责。

### Lines 87-91: Document the class `TestDeepseekR1MXFP4KvFp8PerfMI35x`
```python
    """MI35x Nightly performance benchmark for DeepSeek-R1-MXFP4 with KV Cache FP8.

    Tests the DeepSeek-R1-MXFP4 quantized model on TP=8 with --kv-cache-dtype fp8_e4m3.
    Uses local path if available, otherwise downloads from HuggingFace.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDeepseekR1MXFP4KvFp8PerfMI35x`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDeepseekR1MXFP4KvFp8PerfMI35x`的设计意图。

### Lines 93-122: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = get_model_path()
        print(f"Using model path: {cls.model}")
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.batch_sizes = [1, 8, 16, 64]
        cls.input_lens = tuple(_parse_int_list_env("NIGHTLY_INPUT_LENS", "4096"))
        cls.output_lens = tuple(_parse_int_list_env("NIGHTLY_OUTPUT_LENS", "512"))

        cls.variants = [
            {
                "name": "kv-fp8",
                "other_args": [
                    "--trust-remote-code",
                    "--tp",
                    "8",
                    "--chunked-prefill-size",
                    "131072",
                    "--disable-radix-cache",
                    "--mem-fraction-static",
                    "0.85",
                    "--kv-cache-dtype",
                    "fp8_e4m3",
                ],
            },
        ]

        cls.runner = NightlyBenchmarkRunner(PROFILE_DIR, cls.__name__, cls.base_url)
        cls.runner.setup_profile_directory()
        cls.runner.full_report = f"## {cls.__name__}\n"
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 124-174: Run test: bench one batch
```python
    def test_bench_one_batch(self):
        """Run benchmark across all configured variants."""
        failed_variants = []

        is_local_path = self.model.startswith("/")
        if is_local_path and not os.path.exists(self.model):
            print(f"\n⏭️ SKIPPING: Local model not found at {self.model}")
            self.runner.full_report += (
                f"\n⏭️ Test skipped: Local model not found at {self.model}\n"
            )
            self.runner.write_final_report()
            return

        if is_local_path:
            print(f"📁 Using local model: {self.model}")
        else:
            print(
                f"📥 Using HuggingFace model: {self.model} (will download if not cached)"
            )

        try:
            for variant_config in self.variants:
                with self.subTest(variant=variant_config["name"]):
                    result_tuple = self.runner.run_benchmark_for_model(
                        model_path=self.model,
                        batch_sizes=self.batch_sizes,
                        input_lens=self.input_lens,
                        output_lens=self.output_lens,
                        other_args=variant_config["other_args"],
                        variant=variant_config["name"],
                        extra_bench_args=["--trust-remote-code"],
                        enable_profile=False,
                    )
                    results = result_tuple[0]
                    success = result_tuple[1]

                    if not success:
                        failed_variants.append(variant_config["name"])

                    if results:
                        self.runner.full_report += (
                            generate_simple_markdown_report(results) + "\n"
                        )
        finally:
            self.runner.write_final_report()

        if failed_variants:
            raise AssertionError(
                f"Benchmark failed for {self.model} with the following variants: "
                f"{', '.join(failed_variants)}"
            )
```
**EN:** This test method exercises bench one batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bench one batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 177-178: Expose unittest entrypoint
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
