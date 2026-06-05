# test_input_embeds_chunked.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/input_embedding/test_input_embeds_chunked.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates input embeds chunked behavior in SGLang's input embedding area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 input embedding 领域中与 input embeds chunked 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting statements / 辅助语句
```python
"""Regression tests for input_embeds shape-mismatch bugs.

Covers two bugs with the same crash signature
(RuntimeError: shape mismatch in set_kv_buffer) but opposite polarity:

- Chunked prefill truncation (#20376): PrefillAdder truncates fill_ids and
  extend_input_len on chunk overflow but not input_embeds, so the full array
  flows through while out_cache_loc is sized for the truncated length.
  Polarity: cache_k > loc.

- Retraction with output_ids (#14110): after retraction, fill_ids includes
  accumulated output_ids but input_embeds only covers origin_input_ids.
  Polarity: cache_k < loc.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-31: module imports and dependencies / 模块导入与依赖
```python
import unittest

import requests
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `requests`, `torch`, `transformers`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `requests`, `torch`, `transformers`。

### Lines 33-41: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=43, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=43, suite="stage-b-test-1-gpu-small-amd")

CHUNKED_PREFILL_SIZE = 256

# Shared reference model — loaded once per process, not per test class.
_MODEL = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
_tokenizer = None
_ref_model = None
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 44-48: function load ref / 函数 load ref
```python
def _load_ref():
    global _tokenizer, _ref_model
    if _tokenizer is None:
        _tokenizer = AutoTokenizer.from_pretrained(_MODEL)
        _ref_model = AutoModelForCausalLM.from_pretrained(_MODEL)
```
**EN:** This block implements `_load_ref` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_load_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 51-55: function embeds for / 函数 embeds for
```python
def _embeds_for(text: str) -> list[list[float]]:
    _load_ref()
    ids = _tokenizer(text, return_tensors="pt")["input_ids"]
    embeds = _ref_model.get_input_embeddings()(ids)
    return embeds.squeeze(0).to(torch.float32).tolist()
```
**EN:** This block implements `_embeds_for` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_embeds_for`，承担模块行为中的一个聚焦逻辑片段。

### Lines 58-71: function generate / 函数 generate
```python
def _generate(base_url, input_embeds, max_new_tokens, ignore_eos=False, timeout=120):
    resp = requests.post(
        f"{base_url}/generate",
        json={
            "input_embeds": input_embeds,
            "sampling_params": {
                "temperature": 0,
                "max_new_tokens": max_new_tokens,
                "ignore_eos": ignore_eos,
            },
        },
        timeout=timeout,
    )
    return resp
```
**EN:** This block implements `_generate` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_generate`，承担模块行为中的一个聚焦逻辑片段。

### Lines 74-74: class TestInputEmbedsChunkedAndRetract declaration / 类 TestInputEmbedsChunkedAndRetract 声明
```python
class TestInputEmbedsChunkedAndRetract(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 75-81: supporting statements / 辅助语句
```python
    """Single server launch covering both bugs.

    Both tests require --disable-radix-cache (for input_embeds). The chunked
    prefill test needs a small --chunked-prefill-size. The retraction test
    uses SGLANG_TEST_RETRACT to deterministically force retraction every few
    scheduler iterations regardless of KV pressure.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 83-100: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        # SGLANG_TEST_RETRACT forces retraction periodically; this is
        # deterministic and doesn't require guessing KV budgets.
        with envs.SGLANG_TEST_RETRACT.override(True):
            cls.process = popen_launch_server(
                _MODEL,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--disable-radix-cache",
                    "--chunked-prefill-size",
                    str(CHUNKED_PREFILL_SIZE),
                    "--cuda-graph-max-bs",
                    "4",
                ],
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 102-104: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 106-107: method assert server alive / 方法 assert server alive
```python
    def _assert_server_alive(self):
        self.assertIsNone(self.process.poll(), "server process crashed")
```
**EN:** This block implements `_assert_server_alive` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_server_alive`，承担模块行为中的一个聚焦逻辑片段。

### Lines 109-133: test case chunked prefill truncation and continuation / 测试用例 chunked prefill truncation and continuation
```python
    def test_chunked_prefill_truncation_and_continuation(self):
        """Regression test for #20376.

        A single request longer than chunked_prefill_size deterministically
        exercises both (a) first-chunk truncation and (b) chunk continuation,
        without any concurrent-timing dependency. Pre-fix this crashes in
        set_kv_buffer on both chunks.
        """
        # ~80 tokens each repetition; 6 repetitions exceeds CHUNKED_PREFILL_SIZE
        # comfortably. Token count is model-dependent so assert it.
        text = "The quick brown fox jumps over the lazy dog. " * 40
        embeds = _embeds_for(text)
        self.assertGreater(
            len(embeds),
            CHUNKED_PREFILL_SIZE,
            f"prompt must exceed chunked_prefill_size={CHUNKED_PREFILL_SIZE} "
            f"to trigger chunking; got {len(embeds)} tokens",
        )

        resp = _generate(self.base_url, embeds, max_new_tokens=8)
        self.assertEqual(resp.status_code, 200, resp.text[:300])
        body = resp.json()
        self.assertIn("text", body)
        self.assertIsInstance(body["text"], str)
        self._assert_server_alive()
```
**EN:** Regression test for #20376. This test exercises `test_chunked_prefill_truncation_and_continuation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression test for #20376. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunked_prefill_truncation_and_continuation`。

### Lines 135-157: test case chunked prefill batch truncation / 测试用例 chunked prefill batch truncation
```python
    def test_chunked_prefill_batch_truncation(self):
        """Regression test for #20376 — multi-request batch case.

        A batch POST with total tokens > chunked_prefill_size goes through a
        single ZMQ send, so all requests land in the same scheduler iteration
        and the PrefillAdder is forced to truncate at least one. This matches
        the original thundering-herd trigger without HTTP timing races.
        """
        text = "The quick brown fox jumps over the lazy dog. " * 8
        embeds = _embeds_for(text)
        seq_len = len(embeds)

        # Enough batched requests to overflow the chunk budget.
        n = max(4, CHUNKED_PREFILL_SIZE // seq_len + 2)
        self.assertGreater(n * seq_len, CHUNKED_PREFILL_SIZE)

        resp = _generate(self.base_url, [embeds] * n, max_new_tokens=8)
        self.assertEqual(resp.status_code, 200, resp.text[:300])
        results = resp.json()
        self.assertEqual(len(results), n)
        for r in results:
            self.assertIn("text", r)
        self._assert_server_alive()
```
**EN:** Regression test for #20376 — multi-request batch case. This test exercises `test_chunked_prefill_batch_truncation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression test for #20376 — multi-request batch case. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunked_prefill_batch_truncation`。

### Lines 159-185: test case retraction with output ids / 测试用例 retraction with output ids
```python
    def test_retraction_with_output_ids(self):
        """Regression test for #14110.

        SGLANG_TEST_RETRACT forces retraction every few scheduler iterations.
        Combined with ignore_eos and a reasonable max_new_tokens, at least one
        request is retracted mid-decode with non-empty output_ids, then
        re-prefilled. Pre-#14110 this crashes (cache_k < loc) because fill_ids
        includes output_ids but input_embeds does not.
        """
        text = "The quick brown fox jumps over the lazy dog. " * 4
        embeds = _embeds_for(text)

        # Batch of requests with enough decode steps that SGLANG_TEST_RETRACT
        # (interval=3 by default) fires mid-decode.
        n = 4
        resp = _generate(
            self.base_url,
            [embeds] * n,
            max_new_tokens=32,
            ignore_eos=True,
        )
        self.assertEqual(resp.status_code, 200, resp.text[:300])
        results = resp.json()
        self.assertEqual(len(results), n)
        for r in results:
            self.assertIn("text", r)
        self._assert_server_alive()
```
**EN:** Regression test for #14110. This test exercises `test_retraction_with_output_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Regression test for #14110. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_retraction_with_output_ids`。

### Lines 188-189: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_load_ref`: This block implements `_load_ref` and captures one focused piece of the module's behavior. / 该代码块实现 `_load_ref`，承担模块行为中的一个聚焦逻辑片段。
- `_embeds_for`: This block implements `_embeds_for` and captures one focused piece of the module's behavior. / 该代码块实现 `_embeds_for`，承担模块行为中的一个聚焦逻辑片段。
- `_generate`: This block implements `_generate` and captures one focused piece of the module's behavior. / 该代码块实现 `_generate`，承担模块行为中的一个聚焦逻辑片段。
- `TestInputEmbedsChunkedAndRetract`: Single server launch covering both bugs. / 用于组织相关测试、夹具或辅助方法。
- `TestInputEmbedsChunkedAndRetract.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestInputEmbedsChunkedAndRetract.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestInputEmbedsChunkedAndRetract._assert_server_alive`: This block implements `_assert_server_alive` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_server_alive`，承担模块行为中的一个聚焦逻辑片段。
- `TestInputEmbedsChunkedAndRetract.test_chunked_prefill_truncation_and_continuation`: Regression test for #20376. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunked_prefill_truncation_and_continuation`。
- `TestInputEmbedsChunkedAndRetract.test_chunked_prefill_batch_truncation`: Regression test for #20376 — multi-request batch case. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunked_prefill_batch_truncation`。
- `TestInputEmbedsChunkedAndRetract.test_retraction_with_output_ids`: Regression test for #14110. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_retraction_with_output_ids`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `requests`, `torch`, `transformers`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 189
