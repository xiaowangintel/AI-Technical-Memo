# test_disaggregation_dp_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_disaggregation_dp_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on distributed disaggregation dp attention in SGLang. It drives an evaluation workflow and verifies the resulting quality signals. / 该测试模块用于分析 SGLang 中与 distributed disaggregation dp attention 相关的实现或行为。 它会驱动评测流程，并验证最终得到的质量信号。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Import dependencies
```python
import unittest
from types import SimpleNamespace

from sglang.bench_serving import run_benchmark
from sglang.srt.environ import envs
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST_MLA,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    get_benchmark_args,
    popen_launch_pd_server,
    try_cached_model,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also runs an evaluation workflow to measure model quality and registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会运行评测流程以衡量模型质量，并将该用例注册到 CUDA CI 覆盖范围。

### Lines 19-19: Register CI metadata
```python
register_cuda_ci(est_time=443, stage="base-c", runner_config="8-gpu-h20")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 22-22: Define class TestDisaggregationDPAttention
```python
class TestDisaggregationDPAttention(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationDPAttention` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDPAttention` 测试类，并说明它通过继承承担的职责。

### Lines 23-25: Declare TestDisaggregationDPAttention configuration
```python
    PREFILL_DP_SIZE = 4
    DECODE_DP_SIZE = 4
    LOAD_BALANCE_METHOD = "auto"
```
**EN:** This block defines class-level settings that are shared across the `TestDisaggregationDPAttention` test methods.
**CN:** 该代码块定义了 `TestDisaggregationDPAttention` 各测试方法共享的类级配置。

### Lines 27-43: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Temporarily disable JIT DeepGEMM
        envs.SGLANG_ENABLE_JIT_DEEPGEMM.set(False)

        cls.model = try_cached_model(DEFAULT_MODEL_NAME_FOR_TEST_MLA)

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 45-67: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            str(cls.PREFILL_DP_SIZE),
            "--dp",
            str(cls.PREFILL_DP_SIZE),
            "--enable-dp-attention",
            "--load-balance-method",
            cls.LOAD_BALANCE_METHOD,
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationDPAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationDPAttention` 内部调用，从而让场景结构更清晰。

### Lines 69-93: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            str(cls.DECODE_DP_SIZE),
            "--dp",
            str(cls.DECODE_DP_SIZE),
            "--enable-dp-attention",
            "--base-gpu-id",
            str(cls.PREFILL_DP_SIZE),
            "--load-balance-method",
            cls.LOAD_BALANCE_METHOD,
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationDPAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationDPAttention` 内部调用，从而让场景结构更清晰。

### Lines 95-108: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=1400,
            num_threads=128,
        )
        metrics = run_eval(args)
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["score"], 0.60)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 111-111: Define class TestDisaggregationDPAttentionRoundRobin
```python
class TestDisaggregationDPAttentionRoundRobin(TestDisaggregationDPAttention):
```
**EN:** This declaration introduces the `TestDisaggregationDPAttentionRoundRobin` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDPAttentionRoundRobin` 测试类，并说明它通过继承承担的职责。

### Lines 112-112: Declare TestDisaggregationDPAttentionRoundRobin configuration
```python
    LOAD_BALANCE_METHOD = "round_robin"
```
**EN:** This block defines class-level settings that are shared across the `TestDisaggregationDPAttentionRoundRobin` test methods.
**CN:** 该代码块定义了 `TestDisaggregationDPAttentionRoundRobin` 各测试方法共享的类级配置。

### Lines 115-129: Run test: bench serving
```python
    def test_bench_serving(self):
        args = get_benchmark_args(
            base_url=f"http://{self.base_host}:{self.lb_port}",
            dataset_name="random",
            tokenizer=self.model,
            num_prompts=1000,
            random_input_len=4096,
            random_output_len=1024,
            request_rate=float("inf"),
            max_concurrency=256,
        )
        result = run_benchmark(args)

        self.assertLess(result["mean_tpot_ms"], 20)
        self.assertEqual(result["completed"], 1000)
```
**EN:** This test method exercises bench serving and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bench serving 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 132-132: Define class TestDisaggregationDPAttentionTotalRequests
```python
class TestDisaggregationDPAttentionTotalRequests(TestDisaggregationDPAttention):
```
**EN:** This declaration introduces the `TestDisaggregationDPAttentionTotalRequests` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDPAttentionTotalRequests` 测试类，并说明它通过继承承担的职责。

### Lines 133-136: Declare TestDisaggregationDPAttentionTotalRequests configuration
```python
    LOAD_BALANCE_METHOD = "total_requests"
    test_gsm8k = unittest.skip(
        "Covered by base class; this class targets total_requests path."
    )(TestDisaggregationDPAttention.test_gsm8k)
```
**EN:** This block defines class-level settings that are shared across the `TestDisaggregationDPAttentionTotalRequests` test methods.
**CN:** 该代码块定义了 `TestDisaggregationDPAttentionTotalRequests` 各测试方法共享的类级配置。

### Lines 138-150: Run test: bench serving
```python
    def test_bench_serving(self):
        args = get_benchmark_args(
            base_url=f"http://{self.base_host}:{self.lb_port}",
            dataset_name="random",
            tokenizer=self.model,
            num_prompts=256,
            random_input_len=2048,
            random_output_len=512,
            request_rate=float("inf"),
            max_concurrency=128,
        )
        result = run_benchmark(args)
        self.assertEqual(result["completed"], 256)
```
**EN:** This test method exercises bench serving and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bench serving 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 153-153: Define class TestDisaggregationDPAttentionTotalTokens
```python
class TestDisaggregationDPAttentionTotalTokens(TestDisaggregationDPAttention):
```
**EN:** This declaration introduces the `TestDisaggregationDPAttentionTotalTokens` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDPAttentionTotalTokens` 测试类，并说明它通过继承承担的职责。

### Lines 154-157: Declare TestDisaggregationDPAttentionTotalTokens configuration
```python
    LOAD_BALANCE_METHOD = "total_tokens"
    test_gsm8k = unittest.skip(
        "Covered by base class; this class targets total_tokens path."
    )(TestDisaggregationDPAttention.test_gsm8k)
```
**EN:** This block defines class-level settings that are shared across the `TestDisaggregationDPAttentionTotalTokens` test methods.
**CN:** 该代码块定义了 `TestDisaggregationDPAttentionTotalTokens` 各测试方法共享的类级配置。

### Lines 159-171: Run test: bench serving
```python
    def test_bench_serving(self):
        args = get_benchmark_args(
            base_url=f"http://{self.base_host}:{self.lb_port}",
            dataset_name="random",
            tokenizer=self.model,
            num_prompts=256,
            random_input_len=2048,
            random_output_len=512,
            request_rate=float("inf"),
            max_concurrency=128,
        )
        result = run_benchmark(args)
        self.assertEqual(result["completed"], 256)
```
**EN:** This test method exercises bench serving and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bench serving 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 174-177: Define class TestDisaggregationDPAttentionExternalRouting
```python
@unittest.skip(
    "Skip this test until new testing logic in mini-lb has been updated in docker image."
)
class TestDisaggregationDPAttentionExternalRouting(TestDisaggregationDPAttention):
```
**EN:** This declaration introduces the `TestDisaggregationDPAttentionExternalRouting` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationDPAttentionExternalRouting` 测试类，并说明它通过继承承担的职责。

### Lines 178-186: Document the class `TestDisaggregationDPAttentionExternalRouting`
```python
    """Test external DP rank assignment via mini-lb --test-external-dp-routing.

    NOTE: In PD disaggregation the response comes from the decode server,
    so meta_info["dp_rank"] reflects the decode-side DP rank. Prefill DP
    rank correctness is verified implicitly — if the wrong prefill DP
    worker were used, KV transfer would fail and the request would error.
    The mini-lb internally verifies meta_info["dp_rank"] matches the
    assigned decode dp_rank; a mismatch returns HTTP 500.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDisaggregationDPAttentionExternalRouting`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDisaggregationDPAttentionExternalRouting`的设计意图。

### Lines 188-209: Define helper: launch lb
```python
    @classmethod
    def launch_lb(cls):
        from sglang.test.test_utils import popen_with_error_check

        lb_command = [
            "python3",
            "-m",
            "sglang_router.launch_router",
            "--pd-disaggregation",
            "--mini-lb",
            "--test-external-dp-routing",
            "--prefill",
            cls.prefill_url,
            "--decode",
            cls.decode_url,
            "--host",
            cls.base_host,
            "--port",
            cls.lb_port,
        ]
        cls.process_lb = popen_with_error_check(lb_command)
        cls.wait_server_ready(cls.lb_url + "/health", process=cls.process_lb)
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationDPAttentionExternalRouting` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationDPAttentionExternalRouting` 内部调用，从而让场景结构更清晰。

### Lines 212-213: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.bench_serving`, `sglang.srt.environ`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `run_eval`, `SimpleNamespace`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
