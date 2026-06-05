# test_bench_serving_1gpu_part1.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_bench_serving_1gpu_part1.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates bench serving 1gpu part1 behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 bench serving 1gpu part1 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: supporting statements / 辅助语句
```python
"""
Performance tests for single GPU - LLM throughput/latency and LoRA tests.
Works on 5090 (32GB).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 6-20: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import itertools
import unittest

import requests

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    run_bench_serving,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `itertools`, `unittest`, `requests`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `itertools`, `unittest`, `requests`。

### Lines 22-23: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=1210, stage="extra-a", runner_config="1-gpu-large")
register_amd_ci(est_time=1100, suite="stage-b-test-1-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 26-26: class TestBenchServing1GPUPart1 declaration / 类 TestBenchServing1GPUPart1 声明
```python
class TestBenchServing1GPUPart1(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 27-43: test case offline throughput default / 测试用例 offline throughput default
```python
    def test_offline_throughput_default(self):
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST,
            num_prompts=500,
            request_rate=float("inf"),
            other_server_args=[],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_offline_throughput_default\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 3050)
            else:
                self.assertGreater(res["output_throughput"], 3800)
```
**EN:** This test exercises `test_offline_throughput_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_default`。

### Lines 45-66: test case offline throughput non stream small batch size / 测试用例 offline throughput non stream small batch size
```python
    def test_offline_throughput_non_stream_small_batch_size(self):
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST,
            num_prompts=200,
            request_rate=float("inf"),
            other_server_args=["--max-running-requests", "10"],
            dataset_name="sharegpt",
            random_input_len=None,
            random_output_len=None,
            disable_stream=True,
            need_warmup=True,
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_offline_throughput_non_stream_small_batch_size\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 1000)
            else:
                self.assertGreater(res["output_throughput"], 1050)
```
**EN:** This test exercises `test_offline_throughput_non_stream_small_batch_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_non_stream_small_batch_size`。

### Lines 68-84: test case offline throughput without radix cache / 测试用例 offline throughput without radix cache
```python
    def test_offline_throughput_without_radix_cache(self):
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST,
            num_prompts=500,
            request_rate=float("inf"),
            other_server_args=["--disable-radix-cache"],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_offline_throughput_without_radix_cache\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 3050)
            else:
                self.assertGreater(res["output_throughput"], 3800)
```
**EN:** This test exercises `test_offline_throughput_without_radix_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_without_radix_cache`。

### Lines 86-99: test case offline throughput without chunked prefill / 测试用例 offline throughput without chunked prefill
```python
    def test_offline_throughput_without_chunked_prefill(self):
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST,
            num_prompts=500,
            request_rate=float("inf"),
            other_server_args=["--chunked-prefill-size", "-1"],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_offline_throughput_without_chunked_prefill\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            self.assertGreater(res["output_throughput"], 2600)
```
**EN:** This test exercises `test_offline_throughput_without_chunked_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_without_chunked_prefill`。

### Lines 101-122: test case offline throughput with triton attention backend / 测试用例 offline throughput with triton attention backend
```python
    def test_offline_throughput_with_triton_attention_backend(self):
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST,
            num_prompts=500,
            request_rate=float("inf"),
            other_server_args=[
                "--attention-backend",
                "triton",
                "--context-length",
                "8192",
            ],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_offline_throughput_with_triton_attention_backend\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 3500)
            else:
                self.assertGreater(res["output_throughput"], 3700)
```
**EN:** This test exercises `test_offline_throughput_with_triton_attention_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_with_triton_attention_backend`。

### Lines 124-142: test case online latency default / 测试用例 online latency default
```python
    def test_online_latency_default(self):
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST,
            num_prompts=100,
            request_rate=1,
            other_server_args=[],
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_online_latency_default\n"
                f"median_e2e_latency_ms: {res['median_e2e_latency_ms']:.2f} ms\n"
            )
            self.assertLess(res["median_e2e_latency_ms"], 11000)
            if is_in_amd_ci():
                self.assertLess(res["median_ttft_ms"], 115)
            else:
                self.assertLess(res["median_ttft_ms"], 86)
            self.assertLess(res["median_itl_ms"], 10)
```
**EN:** This test exercises `test_online_latency_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_latency_default`。

### Lines 144-158: test case online lora latency / 测试用例 online lora latency
```python
    def test_online_lora_latency(self):
        res = self._run_lora_latency_test(enable_background_task=False)

        if is_in_ci():
            write_github_step_summary(
                f"### test_online_lora_latency\n"
                f"median_e2e_latency_ms: {res['median_e2e_latency_ms']:.2f} ms\n"
                f"median_ttft_ms: {res['median_ttft_ms']:.2f} ms\n"
            )
            self.assertLess(res["median_e2e_latency_ms"], 2400)
            # relax for mi300x (LoRA TTFT ~2x slower than mi325)
            if is_in_amd_ci():
                self.assertLess(res["median_ttft_ms"], 100)
            else:
                self.assertLess(res["median_ttft_ms"], 58)
```
**EN:** This test exercises `test_online_lora_latency` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_lora_latency`。

### Lines 160-174: test case online lora latency with concurrent adapter updates / 测试用例 online lora latency with concurrent adapter updates
```python
    def test_online_lora_latency_with_concurrent_adapter_updates(self):
        res = self._run_lora_latency_test(enable_background_task=True)

        if is_in_ci():
            write_github_step_summary(
                f"### test_online_lora_latency_with_concurrent_adapter_updates\n"
                f"median_e2e_latency_ms: {res['median_e2e_latency_ms']:.2f} ms\n"
                f"median_ttft_ms: {res['median_ttft_ms']:.2f} ms\n"
            )
            self.assertLess(res["median_e2e_latency_ms"], 4000)
            # relax for mi300x (LoRA TTFT ~2x slower than mi325)
            if is_in_amd_ci():
                self.assertLess(res["median_ttft_ms"], 130)
            else:
                self.assertLess(res["median_ttft_ms"], 80)
```
**EN:** This test exercises `test_online_lora_latency_with_concurrent_adapter_updates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_lora_latency_with_concurrent_adapter_updates`。

### Lines 176-255: method run lora latency test (part 1/2) / 方法 run lora latency test（第 1/2 部分）
```python
    def _run_lora_latency_test(self, enable_background_task: bool):
        """
        Run a latency test for LoRA with the specified background task setting.
        """

        async def lora_loader_unloader_task(
            base_url: str,
            start_event: asyncio.Event,
            stop_event: asyncio.Event,
        ):
            """
            A background task that repeatedly loads and unloads a LoRA adapter.
            """
            await start_event.wait()

            path_cycler = itertools.cycle(
                [
                    "pbevan11/llama-3.1-8b-ocr-correction",
                    "faridlazuarda/valadapt-llama-3.1-8B-it-chinese",
                    "philschmid/code-llama-3-1-8b-text-to-sql-lora",
                ]
            )
            load_url = f"{base_url}/load_lora_adapter"
            unload_url = f"{base_url}/unload_lora_adapter"
            num_updates = 0

            while not stop_event.is_set():
                lora_path = next(path_cycler)
                response = await asyncio.to_thread(
                    requests.post,
                    load_url,
                    json={"lora_name": lora_path, "lora_path": lora_path},
                )
                self.assertTrue(
                    response.ok, f"Failed to load LoRA adapter: {response.text}"
                )
                num_updates += 1

                if stop_event.is_set():
                    break

                await asyncio.sleep(1)

                response = await asyncio.to_thread(
                    requests.post,
                    unload_url,
                    json={"lora_name": lora_path},
                )
                self.assertTrue(
                    response.ok, f"Failed to unload LoRA adapter: {response.text}"
                )
                num_updates += 1

                await asyncio.sleep(1)

        background_task = lora_loader_unloader_task if enable_background_task else None
        res = run_bench_serving(
            model=DEFAULT_MODEL_NAME_FOR_TEST,
            num_prompts=400,
            request_rate=8,
            other_server_args=[
                "--enable-lora",
                "--max-loras-per-batch",
                "1",
                "--disable-radix-cache",
                "--random-seed",
                "42",
                "--mem-fraction-static",
                "0.8",
                "--lora-paths",
                "nvidia/llama-3.1-nemoguard-8b-topic-control",
                "--max-lora-rank",
                "256",
            ],
            dataset_name="random",
            random_input_len=256,
            random_output_len=256,
            lora_name=["nvidia/llama-3.1-nemoguard-8b-topic-control"],
            background_task=background_task,
        )
```
**EN:** Run a latency test for LoRA with the specified background task setting. This block implements `_run_lora_latency_test` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** Run a latency test for LoRA with the specified background task setting. 该代码块实现 `_run_lora_latency_test`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 256-257: method run lora latency test (part 2/2) / 方法 run lora latency test（第 2/2 部分）
```python

        return res
```
**EN:** Run a latency test for LoRA with the specified background task setting. This block implements `_run_lora_latency_test` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** Run a latency test for LoRA with the specified background task setting. 该代码块实现 `_run_lora_latency_test`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 260-261: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBenchServing1GPUPart1`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBenchServing1GPUPart1.test_offline_throughput_default`: This test exercises `test_offline_throughput_default` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_default`。
- `TestBenchServing1GPUPart1.test_offline_throughput_non_stream_small_batch_size`: This test exercises `test_offline_throughput_non_stream_small_batch_size` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_non_stream_small_batch_size`。
- `TestBenchServing1GPUPart1.test_offline_throughput_without_radix_cache`: This test exercises `test_offline_throughput_without_radix_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_without_radix_cache`。
- `TestBenchServing1GPUPart1.test_offline_throughput_without_chunked_prefill`: This test exercises `test_offline_throughput_without_chunked_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_without_chunked_prefill`。
- `TestBenchServing1GPUPart1.test_offline_throughput_with_triton_attention_backend`: This test exercises `test_offline_throughput_with_triton_attention_backend` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_offline_throughput_with_triton_attention_backend`。
- `TestBenchServing1GPUPart1.test_online_latency_default`: This test exercises `test_online_latency_default` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_latency_default`。
- `TestBenchServing1GPUPart1.test_online_lora_latency`: This test exercises `test_online_lora_latency` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_lora_latency`。
- `TestBenchServing1GPUPart1.test_online_lora_latency_with_concurrent_adapter_updates`: This test exercises `test_online_lora_latency_with_concurrent_adapter_updates` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_online_lora_latency_with_concurrent_adapter_updates`。
- `TestBenchServing1GPUPart1._run_lora_latency_test`: Run a latency test for LoRA with the specified background task setting. / 该代码块实现 `_run_lora_latency_test`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `itertools`, `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 261
