# test_return_routed_experts.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_return_routed_experts.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates return routed experts behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 return routed experts 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import json
import logging
import unittest
from typing import List

import aiohttp
import numpy as np
import requests
import torch
from torch.nn.utils.rnn import pad_sequence

from sglang.benchmark.utils import download_and_cache_hf_file
from sglang.srt.state_capturer.routed_experts import (
    extract_routed_experts_from_meta_info,
)
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_ENABLE_ROUTED_EXPERTS_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `json`, `logging`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `json`, `logging`, `unittest`。

### Lines 27-39: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=400, stage="extra-b", runner_config="4-gpu-h100")

# FP8 variant of Qwen3-30B-A3B: required because DeepEP normal/LL fast paths in
# ep_moe/layer.py only run for {Fp8Config (via deep_gemm), W4AFp8Config, aiter,
# NPU, modelopt_fp4+cutedsl}. Bf16 hits an `assert False, "deprecated"` today.
MODEL_PATH = "Qwen/Qwen3-30B-A3B-FP8"

SHAREGPT_REPO_ID = "anon8231489123/ShareGPT_Vicuna_unfiltered"
SHAREGPT_FILENAME = "ShareGPT_V3_unfiltered_cleaned_split.json"
logger = logging.getLogger(__name__)

_QWEN3_30B_A3B_NUM_LAYERS = 48
_QWEN3_30B_A3B_TOPK = 8
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, getLogger.
**CN:** 该代码块通过 register_cuda_ci, getLogger 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 42-42: class TestReturnRoutedExperts declaration / 类 TestReturnRoutedExperts 声明
```python
class TestReturnRoutedExperts(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 43-53: supporting statements / 辅助语句
```python
    """End-to-end check that --enable-return-routed-experts stays correct
    under DeepEP a2a + attn_tp_size > 1, across overlap/cuda-graph/radix
    optimisations.

    Both servers run ``--tp 4 --dp 2 --enable-dp-attention --moe-a2a-backend
    deepep`` so attn_tp_size=2 and the all-gather hot path in
    RoutedExpertsCapturer.capture is hit on every step. Baseline disables
    overlap/cuda-graph/radix to give a deterministic ground truth; reference
    leaves them on. If the gather were skipping a rank or racing against the
    forward stream, the captured topk_ids would diverge between the two.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 55-117: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        common = [
            "--enable-return-routed-experts",
            "--enable-deterministic-inference",
            "--tp",
            4,
            "--dp",
            2,
            "--enable-dp-attention",
            "--moe-a2a-backend",
            "deepep",
            # Force normal-mode dispatch: deepep auto routes decode through
            # low_latency mode whose buffer (num_max_dispatch_tokens_per_rank)
            # is undersized for cuda graph capture at default --cuda-graph-max-bs.
            "--deepep-mode",
            "normal",
        ]
        cls.baseline_args = common + [
            "--disable-overlap-schedule",
            "--disable-cuda-graph",
            "--disable-radix-cache",
        ]
        cls.reference_args = common
        cls.sampling_args = {"temperature": 0}
        # prepare ShareGPT dataset
        dataset_path = download_and_cache_hf_file(SHAREGPT_REPO_ID, SHAREGPT_FILENAME)
        with open(dataset_path) as f:
            data = json.load(f)
        print(f"Dataset size: {len(data)}")
        cls.texts = []
        for s in data:
            if "conversations" in s and len(s["conversations"]) > 0:
                try:
                    text = s["conversations"][0]["value"]
                    if isinstance(text, str) and len(text) <= 2000:
                        cls.texts.append(text)
                except (KeyError, IndexError, TypeError) as e:
                    print(f"Warning: Skipping invalid conversation data: {e}")
                    continue

        if not cls.texts:
            raise ValueError("No valid texts found in the dataset")
        cls.texts = cls.texts[:100]
        cls._endpoints = [
            (
                "/generate",
                cls._build_generate_payload,
                extract_routed_experts_from_meta_info,
            ),
            (
                "/v1/chat/completions",
                cls._build_chat_payload,
                extract_routed_experts_from_openai_response,
            ),
            (
                "/v1/completions",
                cls._build_completion_payload,
                extract_routed_experts_from_openai_response,
            ),
        ]
        cls.baseline_results = cls._collect_results(cls.baseline_args)
        cls.reference_results = cls._collect_results(cls.reference_args)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 119-121: test case return routed experts / 测试用例 return routed experts
```python
    @classmethod
    def test_return_routed_experts(cls):
        cls._run_endpoint_test("/generate")
```
**EN:** This test exercises `test_return_routed_experts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_return_routed_experts`。

### Lines 123-125: test case return routed experts chat completions / 测试用例 return routed experts chat completions
```python
    @classmethod
    def test_return_routed_experts_chat_completions(cls):
        cls._run_endpoint_test("/v1/chat/completions")
```
**EN:** This test exercises `test_return_routed_experts_chat_completions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_return_routed_experts_chat_completions`。

### Lines 127-129: test case return routed experts completions / 测试用例 return routed experts completions
```python
    @classmethod
    def test_return_routed_experts_completions(cls):
        cls._run_endpoint_test("/v1/completions")
```
**EN:** This test exercises `test_return_routed_experts_completions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_return_routed_experts_completions`。

### Lines 131-156: method run endpoint test / 方法 run endpoint test
```python
    @classmethod
    def _run_endpoint_test(cls, endpoint):
        captured_baseline_experts = cls.baseline_results[endpoint]
        captured_reference_experts = cls.reference_results[endpoint]

        check_all_experts_id_valid(captured_baseline_experts)
        check_all_experts_id_valid(captured_reference_experts)

        num_baseline_topks = (
            sum([len(seq) for seq in captured_baseline_experts])
            * len(captured_baseline_experts[0][0])
            * len(captured_baseline_experts[0][0][0])
        )

        num_mismatches = compare_baseline_w_reference(
            captured_baseline_experts, captured_reference_experts
        )
        logger.info(
            f"Total mismatches report: {num_mismatches} out of {num_baseline_topks} ({num_mismatches/num_baseline_topks:.4%})"
        )
        print(
            f"Total mismatches report: {num_mismatches} out of {num_baseline_topks} ({num_mismatches/num_baseline_topks:.4%})"
        )
        assert (
            num_mismatches / num_baseline_topks < 0.10
        ), f"Too many mismatches: {num_mismatches} out of {num_baseline_topks} ({num_mismatches/num_baseline_topks:.4%})"
```
**EN:** This block implements `_run_endpoint_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_endpoint_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 158-172: method collect results / 方法 collect results
```python
    @classmethod
    def _collect_results(
        cls,
        other_args,
    ):
        process = popen_launch_server(
            MODEL_PATH,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
        )
        try:
            return asyncio.run(cls._collect_results_async())
        finally:
            kill_process_tree(process.pid)
```
**EN:** This block implements `_collect_results` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_collect_results`，承担模块行为中的一个聚焦逻辑片段。

### Lines 174-194: method collect results async / 方法 collect results async
```python
    @classmethod
    async def _collect_results_async(cls):
        results = {}
        async with aiohttp.ClientSession() as session:
            for endpoint, payload_builder, response_extractor in cls._endpoints:
                tasks = [
                    asyncio.create_task(
                        make_request(
                            session,
                            f"{DEFAULT_URL_FOR_TEST}{endpoint}",
                            payload_builder(text),
                        )
                    )
                    for text in cls.texts
                ]
                # return value shape: List[[seq_len, num_layers, topk]...]
                http_result = await asyncio.gather(*tasks)
                results[endpoint] = [
                    response_extractor(res).reshape(-1, 48, 8) for res in http_result
                ]
        return results
```
**EN:** This block implements `_collect_results_async` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_collect_results_async`，承担模块行为中的一个聚焦逻辑片段。

### Lines 196-203: method build generate payload / 方法 build generate payload
```python
    @classmethod
    def _build_generate_payload(cls, text):
        return {
            "text": text,
            "sampling_params": cls.sampling_args,
            "return_routed_experts": True,
            "max_new_tokens": 100,
        }
```
**EN:** This block implements `_build_generate_payload` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_generate_payload`，承担模块行为中的一个聚焦逻辑片段。

### Lines 205-212: method build chat payload / 方法 build chat payload
```python
    @classmethod
    def _build_chat_payload(cls, text):
        return {
            "messages": [{"role": "user", "content": text}],
            "temperature": 0,
            "max_tokens": 100,
            "return_routed_experts": True,
        }
```
**EN:** This block implements `_build_chat_payload` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_chat_payload`，承担模块行为中的一个聚焦逻辑片段。

### Lines 214-221: method build completion payload / 方法 build completion payload
```python
    @classmethod
    def _build_completion_payload(cls, text):
        return {
            "prompt": text,
            "temperature": 0,
            "max_tokens": 100,
            "return_routed_experts": True,
        }
```
**EN:** This block implements `_build_completion_payload` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_completion_payload`，承担模块行为中的一个聚焦逻辑片段。

### Lines 224-227: helper routine make request / 辅助流程 make request
```python
async def make_request(session, url, payload):
    """Make a single async HTTP request"""
    async with session.post(url=url, json=payload) as response:
        return await response.json()
```
**EN:** Make a single async HTTP request This helper encapsulates `make_request` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Make a single async HTTP request 该辅助函数封装了 `make_request`，以便周围测试复用准备、执行或校验逻辑。

### Lines 230-242: function extract routed experts from openai response / 函数 extract routed experts from openai response
```python
def extract_routed_experts_from_openai_response(response):
    if "error" in response:
        raise ValueError(f"OpenAI response error: {response['error']}")
    # sglext is at response level (not in choices) as of PR #17648
    sglext = response.get("sglext", None)
    if sglext is None:
        raise ValueError("OpenAI response missing sglext.")
    routed_experts = sglext.get("routed_experts", None)
    if routed_experts is None:
        raise ValueError("OpenAI response sglext missing routed_experts.")
    return extract_routed_experts_from_meta_info(
        {"meta_info": {"routed_experts": routed_experts}}
    )
```
**EN:** This block implements `extract_routed_experts_from_openai_response` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `extract_routed_experts_from_openai_response`，承担模块行为中的一个聚焦逻辑片段。

### Lines 245-253: helper routine check all experts id valid / 辅助流程 check all experts id valid
```python
def check_all_experts_id_valid(experts: List[List[List[int]]]):
    tensor_list = [torch.tensor(lst) for lst in experts]
    padded_tensor = pad_sequence(tensor_list, batch_first=True, padding_value=0)

    # temporary hardcode as we only use Qwen3 30BA3B
    if not ((padded_tensor >= 0) & (padded_tensor <= 127)).all():
        raise ValueError(
            f"Some expert indices are out of valid range [0, 127], MAX: {padded_tensor.max()} MIN: {padded_tensor.min()}"
        )
```
**EN:** This helper encapsulates `check_all_experts_id_valid` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `check_all_experts_id_valid`，以便周围测试复用准备、执行或校验逻辑。

### Lines 256-269: function compare baseline w reference / 函数 compare baseline w reference
```python
def compare_baseline_w_reference(baseline, reference):
    num_total_mismatches = 0
    for baseline_seq, reference_seq in zip(baseline, reference):
        for bsl_token, ref_token in zip(baseline_seq, reference_seq):
            for bsl_topk, ref_topk in zip(bsl_token, ref_token):
                len_bsl, len_ref = len(bsl_topk), len(ref_topk)
                set_bsl, set_ref = set(bsl_topk), set(ref_topk)
                if set_bsl != set_ref:
                    num_total_mismatches += len(set_bsl - set_ref)
                if (len_bsl != len_ref) or (len_bsl != len(set_bsl)):
                    raise ValueError(
                        f"Duplicates experts ids found: Baseline({len_bsl}): {bsl_topk} vs Reference({len_ref}): {ref_topk}"
                    )
    return num_total_mismatches
```
**EN:** This block implements `compare_baseline_w_reference` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `compare_baseline_w_reference`，承担模块行为中的一个聚焦逻辑片段。

### Lines 272-272: class TestRoutedExpertsStartLen declaration / 类 TestRoutedExpertsStartLen 声明
```python
class TestRoutedExpertsStartLen(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 273-280: class-level constants and configuration for `TestRoutedExpertsStartLen` / 类级常量与配置
```python
    """Verify the `routed_experts_start_len` parameter:

    - default (0) returns the full sequence
    - explicit start_len crops the response and the cropped tail matches
      the corresponding tail of the full response
    """

    MAX_NEW_TOKENS = 8
```
**EN:** This block defines shared names such as `MAX_NEW_TOKENS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `MAX_NEW_TOKENS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 282-294: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.process = popen_launch_server(
            DEFAULT_ENABLE_ROUTED_EXPERTS_MODEL_NAME_FOR_TEST,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--enable-return-routed-experts",
                "--enable-deterministic-inference",
                "--tp",
                2,
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 296-299: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 301-305: method send / 方法 send
```python
    def _send(self, payload: dict) -> dict:
        resp = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate", json=payload, timeout=120
        )
        return resp
```
**EN:** This block implements `_send` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_send`，承担模块行为中的一个聚焦逻辑片段。

### Lines 307-318: method build payload / 方法 build payload
```python
    def _build_payload(self, **extra) -> dict:
        payload = {
            "text": "User: Tell me a fact about cats.\nAssistant:",
            "sampling_params": {
                "temperature": 0,
                "max_new_tokens": self.MAX_NEW_TOKENS,
                "ignore_eos": True,
            },
            "return_routed_experts": True,
        }
        payload.update(extra)
        return payload
```
**EN:** This block implements `_build_payload` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_payload`，承担模块行为中的一个聚焦逻辑片段。

### Lines 320-323: method routed experts / 方法 routed experts
```python
    def _routed_experts(self, resp_json: dict):
        return extract_routed_experts_from_meta_info(resp_json).reshape(
            -1, _QWEN3_30B_A3B_NUM_LAYERS, _QWEN3_30B_A3B_TOPK
        )
```
**EN:** This block implements `_routed_experts` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_routed_experts`，承担模块行为中的一个聚焦逻辑片段。

### Lines 325-327: method seqlen / 方法 seqlen
```python
    def _seqlen(self, resp_json: dict) -> int:
        meta = resp_json["meta_info"]
        return meta["prompt_tokens"] + meta["completion_tokens"]
```
**EN:** This block implements `_seqlen` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_seqlen`，承担模块行为中的一个聚焦逻辑片段。

### Lines 329-346: test case start len zero is default / 测试用例 start len zero is default
```python
    def test_start_len_zero_is_default(self):
        """Omitting the field must match `routed_experts_start_len=0`,
        which returns the full sequence (start_len=0)."""
        resp_default = self._send(self._build_payload()).json()
        resp_zero = self._send(self._build_payload(routed_experts_start_len=0)).json()

        rows_default = self._routed_experts(resp_default)
        rows_zero = self._routed_experts(resp_zero)

        seqlen_default = self._seqlen(resp_default)
        seqlen_zero = self._seqlen(resp_zero)
        self.assertEqual(seqlen_default, seqlen_zero)
        self.assertEqual(rows_default.shape[0], seqlen_default - 1)
        self.assertEqual(rows_zero.shape[0], seqlen_zero - 1)
        self.assertTrue(
            np.array_equal(rows_default, rows_zero),
            "default and explicit 0 must produce identical routed experts",
        )
```
**EN:** Omitting the field must match `routed_experts_start_len=0`, which returns the full sequence (start_len=0). This test exercises `test_start_len_zero_is_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Omitting the field must match `routed_experts_start_len=0`, which returns the full sequence (start_len=0). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_len_zero_is_default`。

### Lines 348-375: test case start len controls row count / 测试用例 start len controls row count
```python
    def test_start_len_controls_row_count(self):
        """`routed_experts_start_len=N` must return `seqlen - 1 - N` rows
        and the returned tail must match the corresponding tail of the
        full sequence (start_len omitted)."""
        full_resp = self._send(self._build_payload()).json()
        full_rows = self._routed_experts(full_resp)
        seqlen = self._seqlen(full_resp)
        self.assertEqual(full_rows.shape[0], seqlen - 1)

        start_len = max(1, full_resp["meta_info"]["prompt_tokens"] // 2)

        cropped_resp = self._send(
            self._build_payload(routed_experts_start_len=start_len)
        ).json()
        cropped_rows = self._routed_experts(cropped_resp)
        cropped_seqlen = self._seqlen(cropped_resp)

        self.assertEqual(seqlen, cropped_seqlen)
        expected_rows = seqlen - 1 - start_len
        self.assertEqual(
            cropped_rows.shape[0],
            expected_rows,
            f"expected {expected_rows} rows, got {cropped_rows.shape[0]}",
        )
        self.assertTrue(
            np.array_equal(full_rows[start_len:], cropped_rows),
            "cropped routed experts must match the tail of the full sequence",
        )
```
**EN:** `routed_experts_start_len=N` must return `seqlen - 1 - N` rows and the returned tail must match the corresponding tail of the full sequence (start_len omitted). This test exercises `test_start_len_controls_row_count` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `routed_experts_start_len=N` must return `seqlen - 1 - N` rows and the returned tail must match the corresponding tail of the full sequence (start_len omitted). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_len_controls_row_count`。

### Lines 377-394: test case start len exceeds prompt tokens aborts / 测试用例 start len exceeds prompt tokens aborts
```python
    def test_start_len_exceeds_prompt_tokens_aborts(self):
        """`routed_experts_start_len > prompt_tokens` must abort the request:
        the caller cannot meaningfully reference positions that don't exist
        in the prompt yet."""
        baseline = self._send(self._build_payload()).json()
        prompt_tokens = baseline["meta_info"]["prompt_tokens"]

        ok = self._send(self._build_payload(routed_experts_start_len=prompt_tokens))
        self.assertEqual(
            ok.status_code,
            200,
            f"start_len=={prompt_tokens} should pass, got {ok.text}",
        )

        too_big = self._send(
            self._build_payload(routed_experts_start_len=prompt_tokens + 1)
        )
        self._assert_aborted(too_big, "is higher than the number of input tokens")
```
**EN:** `routed_experts_start_len > prompt_tokens` must abort the request: the caller cannot meaningfully reference positions that don't exist in the prompt yet. This test exercises `test_start_len_exceeds_prompt_tokens_aborts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `routed_experts_start_len > prompt_tokens` must abort the request: the caller cannot meaningfully reference positions that don't exist in the prompt yet. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_len_exceeds_prompt_tokens_aborts`。

### Lines 396-436: test case start len with cache hit / 测试用例 start len with cache hit
```python
    def test_start_len_with_cache_hit(self):
        """`start_len` must allow the radix prefix to extend past it. The
        first request seeds the cache; the second sends the same prompt
        with `start_len` somewhere inside the prompt. We verify:

          - meta_info.cached_tokens > start_len (would be impossible if a
            cap forced the prefix match to <= start_len),
          - the response row count still equals `seqlen - 1 - start_len`.
        """
        cache_salt = "cache-hit-test"
        first = self._send(self._build_payload(extra_key=cache_salt)).json()
        self.assertEqual(
            first["meta_info"].get("cached_tokens", 0),
            0,
            "first request must be a cold miss",
        )

        prompt_tokens = first["meta_info"]["prompt_tokens"]
        start_len = max(1, prompt_tokens // 2)
        second = self._send(
            self._build_payload(
                extra_key=cache_salt,
                routed_experts_start_len=start_len,
            )
        ).json()

        cached = second["meta_info"].get("cached_tokens", 0)
        self.assertGreater(
            cached,
            start_len,
            f"expected radix prefix past start_len={start_len}, "
            f"got cached_tokens={cached} (cap not removed?)",
        )

        rows = self._routed_experts(second)
        expected = self._seqlen(second) - 1 - start_len
        self.assertEqual(
            rows.shape[0],
            expected,
            f"expected {expected} rows, got {rows.shape[0]}",
        )
```
**EN:** `start_len` must allow the radix prefix to extend past it. This test exercises `test_start_len_with_cache_hit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `start_len` must allow the radix prefix to extend past it. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_len_with_cache_hit`。

### Lines 438-459: method assert aborted / 方法 assert aborted
```python
    def _assert_aborted(self, resp, expected_substring: str):
        """Assert a request was aborted with `expected_substring` in the
        error message."""
        if resp.status_code == 200:
            body = resp.json()
            meta = body.get("meta_info", {})
            finish_reason = meta.get("finish_reason") or {}
            message = (
                str(finish_reason.get("message", ""))
                + " "
                + str(body.get("text", ""))
                + " "
                + str(body.get("error", ""))
            )
            self.assertIn(
                expected_substring,
                message,
                f"expected abort with '{expected_substring}', got body={body}",
            )
        else:
            self.assertGreaterEqual(resp.status_code, 400)
            self.assertIn(expected_substring, resp.text)
```
**EN:** Assert a request was aborted with `expected_substring` in the error message. This block implements `_assert_aborted` and captures one focused piece of the module's behavior.
**CN:** Assert a request was aborted with `expected_substring` in the error message. 该代码块实现 `_assert_aborted`，承担模块行为中的一个聚焦逻辑片段。

### Lines 462-463: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestReturnRoutedExperts`: End-to-end check that --enable-return-routed-experts stays correct under DeepEP a2a + attn_tp_size > 1, across overlap/cuda-graph/radix optimisations. / 用于组织相关测试、夹具或辅助方法。
- `make_request`: Make a single async HTTP request / 该辅助函数封装了 `make_request`，以便周围测试复用准备、执行或校验逻辑。
- `extract_routed_experts_from_openai_response`: This block implements `extract_routed_experts_from_openai_response` and captures one focused piece of the module's behavior. / 该代码块实现 `extract_routed_experts_from_openai_response`，承担模块行为中的一个聚焦逻辑片段。
- `check_all_experts_id_valid`: This helper encapsulates `check_all_experts_id_valid` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `check_all_experts_id_valid`，以便周围测试复用准备、执行或校验逻辑。
- `compare_baseline_w_reference`: This block implements `compare_baseline_w_reference` and captures one focused piece of the module's behavior. / 该代码块实现 `compare_baseline_w_reference`，承担模块行为中的一个聚焦逻辑片段。
- `TestRoutedExpertsStartLen`: Verify the `routed_experts_start_len` parameter: - default (0) returns the full sequence - explicit start_len crops the response and the cropped tail matches the corresponding tail of the full response / 用于组织相关测试、夹具或辅助方法。
- `TestReturnRoutedExperts.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestReturnRoutedExperts.test_return_routed_experts`: This test exercises `test_return_routed_experts` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_return_routed_experts`。
- `TestReturnRoutedExperts.test_return_routed_experts_chat_completions`: This test exercises `test_return_routed_experts_chat_completions` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_return_routed_experts_chat_completions`。
- `TestReturnRoutedExperts.test_return_routed_experts_completions`: This test exercises `test_return_routed_experts_completions` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_return_routed_experts_completions`。
- `TestReturnRoutedExperts._run_endpoint_test`: This block implements `_run_endpoint_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_endpoint_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestReturnRoutedExperts._collect_results`: This block implements `_collect_results` and captures one focused piece of the module's behavior. / 该代码块实现 `_collect_results`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `json`, `logging`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `aiohttp`, `numpy`, `requests`, `torch`, `torch.nn.utils.rnn`
- **Internal modules / 内部模块**: `sglang.benchmark.utils`, `sglang.srt.state_capturer.routed_experts`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 463
