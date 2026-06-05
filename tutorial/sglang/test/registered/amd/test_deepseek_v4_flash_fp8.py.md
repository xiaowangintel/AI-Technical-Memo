# test_deepseek_v4_flash_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/test_deepseek_v4_flash_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on amd deepseek v4 flash fp8 in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 amd deepseek v4 flash fp8 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Document the module
```python
"""MI35x DeepSeek-V4-Flash FP8 Test (8-GPU)

Combined accuracy + performance test for DeepSeek-V4-Flash FP8 on MI35x ROCm 7.2.
- Accuracy: GSM8K few-shot eval
- Performance: bench_one_batch_server with input_len=8192, output_len=1024 (bs=1)

Both tests share a single launched server.

Registry: nightly-amd-8-gpu-mi35x-deepseek-v4-flash suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 12-27: Import dependencies
```python
import json
import os
import subprocess
import unittest
from types import SimpleNamespace

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 29-31: Register CI metadata
```python
register_amd_ci(
    est_time=7200, suite="nightly-amd-8-gpu-mi35x-deepseek-v4-flash", nightly=True
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 33-36: Define module constants
```python
DEEPSEEK_V4_FP8_MODEL_PATH = os.environ.get(
    "DEEPSEEK_V4_FP8_MODEL_PATH", "sgl-project/DeepSeek-V4-Flash-FP8"
)
SERVER_LAUNCH_TIMEOUT = 3600
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 40-64: Define module constants
```python
COMMON_ENV_VARS = {
    "SGLANG_OPT_USE_FUSED_COMPRESS": "true",
    "SGLANG_OPT_USE_OLD_COMPRESSOR": "true",
    "SGLANG_OPT_USE_TILELANG_SWA_PREPARE": "false",
    "SGLANG_OPT_USE_TRITON_SWA_PREPARE": "true",
    "SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK": "false",
    "SGLANG_OPT_USE_FUSED_HASH_TOPK": "false",
    "SGLANG_OPT_DEEPGEMM_HC_PRENORM": "false",
    "SGLANG_OPT_USE_TILELANG_MHC_PRE": "false",
    "SGLANG_OPT_USE_AITER_MHC_PRE": "true",
    "SGLANG_OPT_USE_TILELANG_MHC_POST": "false",
    "SGLANG_OPT_USE_AITER_MHC_POST": "true",
    "SGLANG_ENABLE_THINKING": "1",
    "SGLANG_USE_AITER": "1",
    "AITER_BF16_FP8_MOE_BOUND": "1",
    "SGLANG_USE_ROCM700A": "1",
    "SGLANG_FP8_PAGED_MQA_LOGITS_TORCH": "1",
    "SGLANG_OPT_DPSK_V4_RADIX": "0",
    "SGLANG_OPT_USE_OVERLAP_STORE_CACHE": "false",
    "SGLANG_OPT_USE_FUSED_STORE_CACHE": "false",
    "SGLANG_TOPK_TRANSFORM_512_TORCH": "1",
    "SGLANG_OPT_USE_TILELANG_INDEXER": "true",
    "SGLANG_HACK_FLASHMLA_BACKEND": "triton",
    "SGLANG_DSV4_REASONING_EFFORT": "max",
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 67-70: Define module constants
```python
FP8_ENV_VARS = {
    "SGLANG_DSV4_FP4_EXPERTS": "false",
    "SGLANG_FORCE_TRITON_MOE_FP8": "1",
}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 73-73: Define class TestDeepseekV4Fp8
```python
class TestDeepseekV4Fp8(CustomTestCase):
```
**EN:** This declaration introduces the `TestDeepseekV4Fp8` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDeepseekV4Fp8` 测试类，并说明它通过继承承担的职责。

### Lines 74-109: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEEPSEEK_V4_FP8_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST

        env = os.environ.copy()
        env.update(COMMON_ENV_VARS)
        env.update(FP8_ENV_VARS)

        other_args = [
            "--trust-remote-code",
            "--tp",
            "8",
            "--disable-radix-cache",
            "--attention-backend",
            "compressed",
            "--max-running-requests",
            "256",
            "--page-size",
            "256",
            "--chunked-prefill-size",
            "8192",
            "--disable-shared-experts-fusion",
            "--tool-call-parser",
            "deepseekv4",
            "--reasoning-parser",
            "deepseek-v4",
        ]

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=SERVER_LAUNCH_TIMEOUT,
            other_args=other_args,
            env=env,
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 111-113: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 115-134: Run test: a gsm8k
```python
    def test_a_gsm8k(self):
        # `a` prefix to run first (alphabetical) and warm up the server.
        args = SimpleNamespace(
            num_shots=8,
            data_path=None,
            num_questions=1319,
            parallel=1319,
            max_new_tokens=512,
            host="http://127.0.0.1",
            port=int(self.base_url.split(":")[-1]),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"{metrics=}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_gsm8k (deepseek-v4-flash-fp8)\n"
                f'{metrics["accuracy"]=:.3f}\n'
            )
            self.assertGreater(metrics["accuracy"], 0.91)
```
**EN:** This test method exercises a gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 a gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 136-183: Run test: b perf 8k 1k (part 1)
```python
    def test_b_perf_8k_1k(self):
        json_output = "/tmp/deepseek_v4_flash_fp8_perf.json"
        if os.path.exists(json_output):
            os.remove(json_output)

        # First "1" is a warmup; the markdown report below skips it.
        batch_sizes = ["1", "1", "2", "4", "8", "16", "32"]
        cmd = [
            "python3",
            "-m",
            "sglang.bench_one_batch_server",
            "--model",
            "None",
            "--base-url",
            self.base_url,
            "--batch-size",
            *batch_sizes,
            "--input-len",
            "8192",
            "--output-len",
            "1024",
            "--show-report",
            f"--pydantic-result-filename={json_output}",
            "--no-append-to-github-summary",
            "--trust-remote-code",
        ]
        print(f"Running benchmark: {' '.join(cmd)}")
        result = subprocess.run(cmd, capture_output=True, text=True)
        print(result.stdout)
        if result.returncode != 0:
            print(f"STDERR: {result.stderr}")
            self.fail(f"bench_one_batch_server failed (rc={result.returncode})")

        self.assertTrue(
            os.path.exists(json_output),
            f"Benchmark JSON output {json_output} not found",
        )
        with open(json_output) as f:
            results_data = json.load(f)
        self.assertTrue(results_data, "No benchmark results returned")

        if (
            len(results_data) > 1
            and results_data[0]["batch_size"] == results_data[1]["batch_size"]
        ):
            report_results = results_data[1:]
        else:
            report_results = results_data
```
**EN:** This test method exercises b perf 8k 1k and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 b perf 8k 1k 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 185-204: Run test: b perf 8k 1k (part 2)
```python
        summary_lines = [
            "### test_perf_8k_1k (deepseek-v4-flash-fp8)",
            "input_len=8192 output_len=1024",
            "",
            "| batch size | latency (s) | input throughput (tok/s) | output throughput (tok/s) | ITL (ms) |",
            "| ---------- | ----------- | ------------------------ | ------------------------- | -------- |",
        ]
        for r in report_results:
            bs = r["batch_size"]
            latency = r.get("latency", 0.0)
            in_tp = r.get("input_throughput", 0.0)
            out_tp = r.get("output_throughput", 0.0)
            itl = 1 / (out_tp / bs) * 1000 if out_tp > 0 else float("inf")
            summary_lines.append(
                f"| {bs} | {latency:.2f} | {in_tp:.2f} | {out_tp:.2f} | {itl:.2f} |"
            )
            print(
                f"bs={bs} latency={latency:.2f}s "
                f"in_tp={in_tp:.2f} tok/s out_tp={out_tp:.2f} tok/s ITL={itl:.2f}ms"
            )
```
**EN:** This test method exercises b perf 8k 1k and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 b perf 8k 1k 场景，并验证观测到的行为是否符合预期契约。

### Lines 206-207: Run test: b perf 8k 1k (part 3)
```python
        if is_in_ci():
            write_github_step_summary("\n".join(summary_lines) + "\n")
```
**EN:** This test method exercises b perf 8k 1k and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 b perf 8k 1k 场景，并验证观测到的行为是否符合预期契约。

### Lines 210-219: Expose unittest entrypoint
```python
if __name__ == "__main__":
    # run_suite.py's run_one_file launches each test file with `python3 <file> -f`,
    # which enables unittest fail-fast. For this file, `test_a_gsm8k` (accuracy)
    # and `test_b_perf_8k_1k` (performance) are independent measurements that
    # share a very expensive server launch in setUpClass; we want perf data even
    # if accuracy fails. Strip `-f` locally so subsequent test methods still run.
    import sys

    sys.argv = [a for a in sys.argv if a not in ("-f", "--failfast")]
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
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `json`, `os`, `subprocess`, `sys`, `types`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `unittest.main`
