# test_step3p5_flash_chain_mtp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/8-gpu-models/test_step3p5_flash_chain_mtp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on 8 gpu models step3p5 flash chain mtp in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 8 gpu models step3p5 flash chain mtp 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Import dependencies
```python
import unittest
from types import SimpleNamespace

import numpy as np
import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 20-20: Register CI metadata
```python
register_cuda_ci(est_time=480, stage="extra-b", runner_config="8-gpu-h200")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 22-22: Define module constants
```python
STEP3P5_FLASH_MODEL_PATH = "stepfun-ai/Step-3.5-Flash"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 25-25: Define class TestStep3p5FlashChainMTP
```python
class TestStep3p5FlashChainMTP(CustomTestCase):
```
**EN:** This declaration introduces the `TestStep3p5FlashChainMTP` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestStep3p5FlashChainMTP` 测试类，并说明它通过继承承担的职责。

### Lines 26-32: Document the class `TestStep3p5FlashChainMTP`
```python
    """Chain-style multi-layer EAGLE speculative decoding on Step-3.5-Flash.

    Step3p5ForCausalLM auto-enables multi-layer EAGLE and spec v2 when
    --speculative-algorithm=EAGLE is set.  The chain MTP propagation
    (each MTP layer feeds its hidden states to the next) is activated
    automatically for the Step3p5MTP draft architecture.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestStep3p5FlashChainMTP`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestStep3p5FlashChainMTP`的设计意图。

### Lines 34-66: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = STEP3P5_FLASH_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--tp",
            "8",
            "--trust-remote-code",
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-num-steps",
            "3",
            "--speculative-eagle-topk",
            "1",
            "--speculative-num-draft-tokens",
            "4",
            "--attention-backend",
            "fa3",
            "--enable-multi-layer-eagle",
            "--mem-fraction-static",
            "0.75",
            "--chunked-prefill-size",
            "4096",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true, "num_threads": 64}',
        ]
        with envs.SGLANG_ENABLE_SPEC_V2.override(True):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH * 3,
                other_args=other_args,
            )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 68-71: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 73-102: Run test: gsm8k
```python
    def test_gsm8k(self):
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=200,
            num_threads=128,
        )
        metrics = run_eval(args)
        print(f"{metrics=}")

        server_info = requests.get(self.base_url + "/server_info")
        avg_spec_accept_length = server_info.json()["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")
        print(f"{metrics=}")

        if is_in_ci():
            write_github_step_summary(
                f"### test_gsm8k (step-3.5-flash chain mtp)\n"
                f'{metrics["score"]=:.3f}\n'
                f"{avg_spec_accept_length=:.2f}\n"
            )
            self.assertGreater(metrics["score"], 0.83)
            self.assertGreater(avg_spec_accept_length, 2.6)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 104-161: Run test: logprob spec v2 match (part 1)
```python
    def test_logprob_spec_v2_match(self):
        """Verify spec v2 decode logprobs match prefill scoring logprobs.

        Generate tokens with chain MTP spec v2, then score the same sequence
        via prefill-only (no speculation). The two sets of logprobs should be
        close, validating that spec v2 + multi-layer EAGLE computes logprobs
        correctly.
        """
        requests.get(self.base_url + "/flush_cache")

        top_k = 5
        probe_token_ids = [1, 2, 10, 100, 1000]
        prompts = [
            "The capital of France is",
            "Explain quantum computing in simple terms:",
        ]

        for round_idx, prompt in enumerate(prompts):
            with self.subTest(round=round_idx, prompt=prompt):
                gen_res = requests.post(
                    self.base_url + "/generate",
                    json={
                        "text": prompt,
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": 32,
                            "ignore_eos": True,
                        },
                        "return_logprob": True,
                        "top_logprobs_num": top_k,
                        "token_ids_logprob": probe_token_ids,
                        "logprob_start_len": 0,
                    },
                ).json()

                decode_logprobs = gen_res["meta_info"]["output_token_logprobs"]
                decode_top_logprobs = gen_res["meta_info"]["output_top_logprobs"]
                decode_tid_logprobs = gen_res["meta_info"]["output_token_ids_logprobs"]
                input_token_ids = [
                    t[1] for t in gen_res["meta_info"]["input_token_logprobs"]
                ]
                output_token_ids = [t[1] for t in decode_logprobs]
                num_prompt_tokens = gen_res["meta_info"]["prompt_tokens"]

                score_res = requests.post(
                    self.base_url + "/generate",
                    json={
                        "input_ids": input_token_ids + output_token_ids,
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": 0,
                        },
                        "return_logprob": True,
                        "top_logprobs_num": top_k,
                        "token_ids_logprob": probe_token_ids,
                        "logprob_start_len": 0,
                    },
                ).json()
```
**EN:** This test method exercises logprob spec v2 match and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and issues HTTP POST requests against the exposed endpoint.
**CN:** 该测试方法会执行 logprob spec v2 match 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并向暴露的端点发起 HTTP POST 请求。

### Lines 163-207: Run test: logprob spec v2 match (part 2)
```python
                score_logprobs = score_res["meta_info"]["input_token_logprobs"][
                    num_prompt_tokens:
                ]
                score_top_logprobs = score_res["meta_info"]["input_top_logprobs"][
                    num_prompt_tokens:
                ]
                score_tid_logprobs = score_res["meta_info"]["input_token_ids_logprobs"][
                    num_prompt_tokens:
                ]

                self.assertEqual(len(decode_logprobs), len(score_logprobs))

                decode_vals = np.array([t[0] for t in decode_logprobs])
                score_vals = np.array([t[0] for t in score_logprobs])
                max_diff = np.max(np.abs(decode_vals - score_vals))
                print(
                    f"[round {round_idx}] prompt={prompt!r} "
                    f"logprob max_diff={max_diff:.6f}"
                )
                print(f"[round {round_idx}] decode_vals[-5:]={decode_vals[-5:]}")
                print(f"[round {round_idx}] score_vals[-5:]={score_vals[-5:]}")
                self.assertLess(max_diff, 0.255)

                # Top-k / probe tokens are not sampled, so they drift more than
                # the chosen-token logprob under TP=8 + multi-layer EAGLE noise.
                # Collect the diff distribution to see whether outliers are
                # isolated tail tokens or systemic drift before asserting.
                top_diffs = []
                for pos in range(len(decode_logprobs)):
                    dec_top = {t[1]: t[0] for t in decode_top_logprobs[pos]}
                    scr_top = {t[1]: t[0] for t in score_top_logprobs[pos]}
                    common_ids = set(dec_top.keys()) & set(scr_top.keys())
                    self.assertGreater(len(common_ids), 0)
                    for tid in common_ids:
                        top_diffs.append(abs(dec_top[tid] - scr_top[tid]))
                top_diffs_arr = np.array(top_diffs)
                print(
                    f"[round {round_idx}] top-k diffs: "
                    f"n={len(top_diffs_arr)} "
                    f"max={top_diffs_arr.max():.4f} "
                    f"p99={np.percentile(top_diffs_arr, 99):.4f} "
                    f"p95={np.percentile(top_diffs_arr, 95):.4f} "
                    f"p50={np.percentile(top_diffs_arr, 50):.4f} "
                    f"mean={top_diffs_arr.mean():.4f}"
                )
```
**EN:** This test method exercises logprob spec v2 match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 logprob spec v2 match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 209-226: Run test: logprob spec v2 match (part 3)
```python
                self.assertEqual(len(decode_tid_logprobs), len(score_tid_logprobs))
                tid_diffs = []
                for pos in range(len(decode_tid_logprobs)):
                    dec_tid = {t[1]: t[0] for t in decode_tid_logprobs[pos]}
                    scr_tid = {t[1]: t[0] for t in score_tid_logprobs[pos]}
                    self.assertEqual(set(dec_tid.keys()), set(scr_tid.keys()))
                    for tid in dec_tid:
                        tid_diffs.append(abs(dec_tid[tid] - scr_tid[tid]))
                tid_diffs_arr = np.array(tid_diffs)
                print(
                    f"[round {round_idx}] token_ids_logprob diffs: "
                    f"n={len(tid_diffs_arr)} "
                    f"max={tid_diffs_arr.max():.4f} "
                    f"p99={np.percentile(tid_diffs_arr, 99):.4f} "
                    f"p95={np.percentile(tid_diffs_arr, 95):.4f} "
                    f"p50={np.percentile(tid_diffs_arr, 50):.4f} "
                    f"mean={tid_diffs_arr.mean():.4f}"
                )
```
**EN:** This test method exercises logprob spec v2 match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 logprob spec v2 match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 228-237: Run test: logprob spec v2 match (part 4)
```python
                # Bulk of the distribution must stay tight. Tail (max / p99) is
                # dominated by very low-probability tokens whose logprobs are
                # extremely sensitive to BF16 + TP=8 logsumexp noise — a real
                # bug in chain MTP hidden state propagation would shift the
                # median, not just the tail.
                self.assertLess(np.percentile(top_diffs_arr, 50), 0.1)
                self.assertLess(top_diffs_arr.mean(), 0.2)
                self.assertLess(np.percentile(top_diffs_arr, 95), 0.4)
                self.assertLess(np.percentile(tid_diffs_arr, 50), 0.2)
                self.assertLess(tid_diffs_arr.mean(), 0.4)
```
**EN:** This test method exercises logprob spec v2 match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 logprob spec v2 match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 240-241: Expose unittest entrypoint
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
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `numpy`, `requests`, `types`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `CustomTestCase`, `SimpleNamespace`, `requests.get`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
