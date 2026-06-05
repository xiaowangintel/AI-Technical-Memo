# test_bench_serving_1gpu_part2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/perf/test_bench_serving_1gpu_part2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates bench serving 1gpu part2 behavior in SGLang's perf area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 perf 领域中与 bench serving 1gpu part2 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: supporting statements / 辅助语句
```python
"""
Performance tests for single GPU - VLM, Score API, and Embeddings API tests.
Works on 5090 (32GB).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 6-20: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST_SCORE,
    DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
    CustomTestCase,
    is_in_amd_ci,
    is_in_ci,
    run_bench_serving,
    run_embeddings_benchmark,
    run_score_benchmark,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 22-23: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=968, stage="extra-a", runner_config="1-gpu-large")
register_amd_ci(est_time=900, suite="stage-b-test-1-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 26-26: class TestBenchServing1GPUPart2 declaration / 类 TestBenchServing1GPUPart2 声明
```python
class TestBenchServing1GPUPart2(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 27-48: test case vlm offline throughput / 测试用例 vlm offline throughput
```python
    def test_vlm_offline_throughput(self):
        res = run_bench_serving(
            model=DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
            num_prompts=200,
            request_rate=float("inf"),
            other_server_args=[
                "--mem-fraction-static",
                "0.7",
            ],
            dataset_name="mmmu",
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_vlm_offline_throughput\n"
                f"Output throughput: {res['output_throughput']:.2f} token/s\n"
            )
            # relax for mi300x
            if is_in_amd_ci():
                self.assertGreater(res["output_throughput"], 900)
            else:
                self.assertGreater(res["output_throughput"], 2500)
```
**EN:** This test exercises `test_vlm_offline_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_offline_throughput`。

### Lines 50-72: test case vlm online latency / 测试用例 vlm online latency
```python
    def test_vlm_online_latency(self):
        res = run_bench_serving(
            model=DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
            num_prompts=250,
            request_rate=1,
            other_server_args=[
                "--mem-fraction-static",
                "0.7",
            ],
            dataset_name="mmmu",
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_vlm_online_latency\n"
                f"median_e2e_latency_ms: {res['median_e2e_latency_ms']:.2f} ms\n"
            )
            self.assertLess(res["median_e2e_latency_ms"], 16500)
            if is_in_amd_ci():
                self.assertLess(res["median_ttft_ms"], 150)
            else:
                self.assertLess(res["median_ttft_ms"], 100)
            self.assertLess(res["median_itl_ms"], 8)
```
**EN:** This test exercises `test_vlm_online_latency` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_online_latency`。

### Lines 74-96: test case score api latency throughput / 测试用例 score api latency throughput
```python
    def test_score_api_latency_throughput(self):
        """Test score API latency and throughput performance"""
        res = run_score_benchmark(
            model=DEFAULT_SMALL_MODEL_NAME_FOR_TEST_SCORE,
            num_requests=1000,
            batch_size=10,
            other_server_args=[],
            need_warmup=True,
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_score_api_throughput\n"
                f"Average latency: {res['avg_latency_ms']:.2f} ms\n"
                f"P95 latency: {res['p95_latency_ms']:.2f} ms\n"
                f"Score API throughput: {res['throughput']:.2f} req/s\n"
                f"Successful requests: {res['successful_requests']}/{res['total_requests']}\n"
            )

        self.assertEqual(res["successful_requests"], res["total_requests"])
        self.assertLess(res["avg_latency_ms"], 48)
        self.assertLess(res["p95_latency_ms"], 50)
        self.assertGreater(res["throughput"], 20)
```
**EN:** Test score API latency and throughput performance This test exercises `test_score_api_latency_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test score API latency and throughput performance 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_api_latency_throughput`。

### Lines 98-131: test case score api batch scaling / 测试用例 score api batch scaling
```python
    def test_score_api_batch_scaling(self):
        """Test score API performance with different batch sizes"""
        batch_sizes = [10, 25, 50]

        for batch_size in batch_sizes:
            res = run_score_benchmark(
                model=DEFAULT_SMALL_MODEL_NAME_FOR_TEST_SCORE,
                num_requests=500,
                batch_size=batch_size,
            )

            if is_in_ci():
                write_github_step_summary(
                    f"### test_score_api_batch_scaling_size_{batch_size}\n"
                    f"Batch size: {batch_size}\n"
                    f"Average latency: {res['avg_latency_ms']:.2f} ms\n"
                    f"P95 latency: {res['p95_latency_ms']:.2f} ms\n"
                    f"Throughput: {res['throughput']:.2f} req/s\n"
                    f"Successful requests: {res['successful_requests']}/{res['total_requests']}\n"
                )

            self.assertEqual(res["successful_requests"], res["total_requests"])
            # relax for mi300x
            if is_in_amd_ci():
                bounds = {10: (60, 65), 25: (70, 80), 50: (80, 90)}
                default_bounds = (90, 90)
            else:
                bounds = {10: (45, 50), 25: (50, 60), 50: (60, 65)}
                default_bounds = (60, 65)
            avg_latency_bound, p95_latency_bound = bounds.get(
                batch_size, default_bounds
            )
            self.assertLess(res["avg_latency_ms"], avg_latency_bound)
            self.assertLess(res["p95_latency_ms"], p95_latency_bound)
```
**EN:** Test score API performance with different batch sizes This test exercises `test_score_api_batch_scaling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test score API performance with different batch sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_api_batch_scaling`。

### Lines 133-162: test case embeddings api latency throughput / 测试用例 embeddings api latency throughput
```python
    def test_embeddings_api_latency_throughput(self):
        """Test embeddings API latency and throughput performance"""
        res = run_embeddings_benchmark(
            model=DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST,
            num_requests=1000,
            batch_size=1,
            input_tokens=500,
            other_server_args=[],
            need_warmup=True,
        )

        if is_in_ci():
            write_github_step_summary(
                f"### test_embeddings_api_throughput\n"
                f"Average latency: {res['avg_latency_ms']:.2f} ms\n"
                f"P95 latency: {res['p95_latency_ms']:.2f} ms\n"
                f"Embeddings API throughput: {res['throughput']:.2f} req/s\n"
                f"Successful requests: {res['successful_requests']}/{res['total_requests']}\n"
            )

        self.assertEqual(res["successful_requests"], res["total_requests"])
        # relax for mi300x
        if is_in_amd_ci():
            self.assertLess(res["avg_latency_ms"], 35)
            self.assertLess(res["p95_latency_ms"], 40)
            self.assertGreater(res["throughput"], 30)
        else:
            self.assertLess(res["avg_latency_ms"], 20)
            self.assertLess(res["p95_latency_ms"], 25)
            self.assertGreater(res["throughput"], 60)
```
**EN:** Test embeddings API latency and throughput performance This test exercises `test_embeddings_api_latency_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embeddings API latency and throughput performance 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embeddings_api_latency_throughput`。

### Lines 164-198: test case embeddings api batch scaling / 测试用例 embeddings api batch scaling
```python
    def test_embeddings_api_batch_scaling(self):
        """Test embeddings API performance with different batch sizes"""
        batch_sizes = [10, 25, 50]

        for batch_size in batch_sizes:
            res = run_embeddings_benchmark(
                model=DEFAULT_SMALL_EMBEDDING_MODEL_NAME_FOR_TEST,
                num_requests=500,
                batch_size=batch_size,
                input_tokens=500,
            )

            if is_in_ci():
                write_github_step_summary(
                    f"### test_embeddings_api_batch_scaling_size_{batch_size}\n"
                    f"Batch size: {batch_size}\n"
                    f"Average latency: {res['avg_latency_ms']:.2f} ms\n"
                    f"P95 latency: {res['p95_latency_ms']:.2f} ms\n"
                    f"Throughput: {res['throughput']:.2f} req/s\n"
                    f"Successful requests: {res['successful_requests']}/{res['total_requests']}\n"
                )

            self.assertEqual(res["successful_requests"], res["total_requests"])
            # relax for mi300x
            if is_in_amd_ci():
                bounds = {10: (80, 90), 25: (140, 150), 50: (230, 240)}
                default_bounds = (300, 300)
            else:
                bounds = {10: (60, 65), 25: (115, 120), 50: (190, 195)}
                default_bounds = (250, 250)
            avg_latency_bound, p95_latency_bound = bounds.get(
                batch_size, default_bounds
            )
            self.assertLess(res["avg_latency_ms"], avg_latency_bound)
            self.assertLess(res["p95_latency_ms"], p95_latency_bound)
```
**EN:** Test embeddings API performance with different batch sizes This test exercises `test_embeddings_api_batch_scaling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embeddings API performance with different batch sizes 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embeddings_api_batch_scaling`。

### Lines 201-202: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBenchServing1GPUPart2`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBenchServing1GPUPart2.test_vlm_offline_throughput`: This test exercises `test_vlm_offline_throughput` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_offline_throughput`。
- `TestBenchServing1GPUPart2.test_vlm_online_latency`: This test exercises `test_vlm_online_latency` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_vlm_online_latency`。
- `TestBenchServing1GPUPart2.test_score_api_latency_throughput`: Test score API latency and throughput performance / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_api_latency_throughput`。
- `TestBenchServing1GPUPart2.test_score_api_batch_scaling`: Test score API performance with different batch sizes / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_api_batch_scaling`。
- `TestBenchServing1GPUPart2.test_embeddings_api_latency_throughput`: Test embeddings API latency and throughput performance / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embeddings_api_latency_throughput`。
- `TestBenchServing1GPUPart2.test_embeddings_api_batch_scaling`: Test embeddings API performance with different batch sizes / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embeddings_api_batch_scaling`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 202
