# test_logprobs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/sample/test_logprobs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `logprobs` behavior and regressions in the v1 stack. / 验证 v1 栈中 `对数概率` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-25)
```python
import itertools
import math
from collections.abc import Generator
from typing import get_args

import pytest
import torch

from tests.utils import large_gpu_mark
from tests.v1.sample.utils import (
    BatchLogprobsComposition,
    BatchLogprobsSpecType,
    assert_incr_detok_str_matches_non_incr_detok_str,
    compute_correct_cumulative_logprob,
    get_test_batch,
)
from vllm import SamplingParams
from vllm.config.model import LogprobsMode
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.platforms import current_platform

from ...conftest import HfRunner, VllmRunner
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm, vllm.config.model, vllm.distributed, vllm.platforms, vllm.v1.engine.logprobs`. Local helpers come from `tests.utils, tests.v1.sample.utils, tests.conftest`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm, vllm.config.model, vllm.distributed, vllm.platforms, vllm.v1.engine.logprobs`。 本地测试辅助逻辑来自 `tests.utils, tests.v1.sample.utils, tests.conftest`。

### Module state / 模块级状态 (lines 27-47)
```python
MODEL = "meta-llama/Llama-3.2-1B-Instruct"
DTYPE = "half"

NONE = BatchLogprobsComposition.NONE
SAMPLE = BatchLogprobsComposition.SAMPLE
PROMPT = BatchLogprobsComposition.PROMPT
SAMPLE_PROMPT = BatchLogprobsComposition.SAMPLE_PROMPT

# On ROCm, floating-point reductions in attention and GEMM kernels are
# non-associative and sensitive to batch geometry. If the ref LLM and
# spec-decode LLM use different scheduling or batch geometry, they can
# follow different reduction orders and produce numerically divergent
# logprobs that get misattributed to spec-decode incorrectness.
#
# Force LLM instances into an identical, deterministic execution
# mode so the test isolates spec-decode correctness only:
ROCM_DETERMINISM_KWARGS: dict = (
    dict(max_num_seqs=1, attention_backend="TRITON_ATTN")
    if current_platform.is_rocm()
    else {}
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MODEL, DTYPE, NONE, SAMPLE, PROMPT, SAMPLE_PROMPT, ...`. Shared setup calls include `current_platform.is_rocm, dict`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MODEL, DTYPE, NONE, SAMPLE, PROMPT, SAMPLE_PROMPT, ...`。 共享初始化调用包括 `current_platform.is_rocm, dict`。

### vllm_model (lines 55-72)
```python
def vllm_model(vllm_runner, request) -> Generator[VllmRunner, None, None]:
    with vllm_runner(
        MODEL,
        dtype=DTYPE,
        max_logprobs=7,
        # Very small number of batched tokens to ensure
        # that we test chunking.
        max_num_batched_tokens=16,
        max_num_seqs=16,
        max_model_len=128,
        enable_chunked_prefill=True,
        enforce_eager=True,
        # TODO: enable this once we support it for
        # prompt logprobs.
        enable_prefix_caching=request.param,
        gpu_memory_utilization=0.4,
    ) as vllm_model:
        yield vllm_model
```
**EN:** Fixture/helper `vllm_model` prepares reusable state for downstream tests. Inputs: `vllm_runner, request`. Key calls include `pytest.fixture, vllm_runner`.
**CN:** `vllm_model` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`vllm_runner, request`。 关键调用包括 `pytest.fixture, vllm_runner`。

### hf_model (lines 76-78)
```python
def hf_model(hf_runner) -> Generator[HfRunner, None, None]:
    with hf_runner(MODEL, dtype=DTYPE) as hf_model:
        yield hf_model
```
**EN:** Fixture/helper `hf_model` prepares reusable state for downstream tests. Inputs: `hf_runner`. Key calls include `pytest.fixture, hf_runner`.
**CN:** `hf_model` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`hf_runner`。 关键调用包括 `pytest.fixture, hf_runner`。

### _repeat_logprob_config (lines 81-126)
```python
def _repeat_logprob_config(
    test_prompts,
    logprob_prompt_logprob_list: BatchLogprobsSpecType,
) -> BatchLogprobsSpecType:
    """Ensure each test prompt has a logprob config.

    A logprob config specifies the optional (i.e.
    may-be-`None`) number of sample logprobs and
    the optional number of prompt logprobs.
    If more test prompts than logprob configs are
    provided, the provided logprob configs are
    tiled to match the number of test prompts.
    If fewer test prompts than logprob configs
    are provided, the list of logprob configs
    is truncated to match the number of test
    prompts.
    # ... excerpt omitted for brevity ...
    logprob_prompt_logprob_list = list(
        itertools.islice(itertools.cycle(logprob_prompt_logprob_list), num_test_prompts)
    )
    # Now the number of prompts should match the number of sample params combos
    assert num_test_prompts == len(logprob_prompt_logprob_list)
    return logprob_prompt_logprob_list
```
**EN:** Helper function `_repeat_logprob_config` encapsulates reusable logic for `repeat logprob config`. Inputs: `test_prompts, logprob_prompt_logprob_list`. Key calls include `len, list, itertools.islice, itertools.cycle`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_repeat_logprob_config` 封装了与 `repeat logprob config` 相关的可复用逻辑。 输入参数：`test_prompts, logprob_prompt_logprob_list`。 关键调用包括 `len, list, itertools.islice, itertools.cycle`。 其中包含 1 个内部断言，用于保护前置假设。

### _run_and_validate (lines 129-289)
```python
def _run_and_validate(
    vllm_model: VllmRunner,
    test_prompts: list[str],
    vllm_sampling_params: SamplingParams,
    hf_logprobs: list[list[torch.Tensor]],
    hf_outputs: list[tuple[list[int], str]],
    logprob_prompt_logprob_list: BatchLogprobsSpecType,
    temperature: float,
    max_tokens: int,
    do_apc: bool,
) -> None:
    vllm_results = vllm_model.llm.generate(
        test_prompts, sampling_params=vllm_sampling_params
    )

    for vllm_result, hf_logprob, hf_output, logprob_prompt_logprob in zip(
        vllm_results, hf_logprobs, hf_outputs, logprob_prompt_logprob_list
    ):
    # ... excerpt omitted for brevity ...
            assert (
            assert num_top_logprobs is not None
            assert vllm_result.outputs[0].logprobs is not None
            assert len(vllm_result.outputs[0].logprobs) == max_tokens
                assert logprobs is not None
                        hf_logprob[0][i][token_id].item(),
                        atol=2e-2,
                        rtol=2e-2,
                    )
        else:
            assert vllm_result.prompt_logprobs is None
```
**EN:** Helper function `_run_and_validate` encapsulates reusable logic for `run and validate`. Inputs: `vllm_model, test_prompts, vllm_sampling_params, hf_logprobs, hf_outputs, logprob_prompt_logprob_list, temperature, max_tokens, ...`. Key calls include `llm.generate, zip, join, assert_incr_detok_str_matches_non_incr_detok_str, enumerate, testing.assert_close`. It includes 21 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_run_and_validate` 封装了与 `run and validate` 相关的可复用逻辑。 输入参数：`vllm_model, test_prompts, vllm_sampling_params, hf_logprobs, hf_outputs, logprob_prompt_logprob_list, temperature, max_tokens, ...`。 关键调用包括 `llm.generate, zip, join, assert_incr_detok_str_matches_non_incr_detok_str, enumerate, testing.assert_close`。 其中包含 21 个内部断言，用于保护前置假设。

### test_get_logprobs_and_prompt_logprobs (lines 296-376)
```python
def test_get_logprobs_and_prompt_logprobs(
    hf_model,
    vllm_model,
    batch_logprobs_composition: BatchLogprobsComposition,
    temperature: float,
    example_prompts: list[str],
) -> None:
    """Test V1 Engine logprobs & prompt logprobs

    Exercise a variety of combinations of `logprobs` and `prompt_logprobs`
    settings and validate that
    * The generated logprobs and prompt logprobs are consistent with the
      configuration settings, in terms of whether or not the logprobs
      (of either type) were requested and how many were requested
    * The generated logprobs are consistent with the generated tokens
    * The generated (prompt)logprobs are consistent with HuggingFace
      (prompt)logprobs, as a reference
    # ... excerpt omitted for brevity ...
            hf_outputs=hf_outputs,
            logprob_prompt_logprob_list=logprob_prompt_logprob_list,
            temperature=temperature,
            max_tokens=max_tokens,
            do_apc=do_apc,
        )
```
**EN:** Parameterized test covering `get logprobs and prompt logprobs`. Parameter axes: `batch_logprobs_composition, temperature`. Inputs/fixtures: `hf_model, vllm_model, batch_logprobs_composition, temperature, example_prompts`. It exercises `mark.parametrize, hf_model.generate_greedy, hf_model.generate_greedy_logprobs, get_test_batch, _repeat_logprob_config, range`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `get 对数概率 and prompt 对数概率` 的测试用例。 参数维度：`batch_logprobs_composition, temperature`。 输入或 fixture：`hf_model, vllm_model, batch_logprobs_composition, temperature, example_prompts`。 该测试会调用 `mark.parametrize, hf_model.generate_greedy, hf_model.generate_greedy_logprobs, get_test_batch, _repeat_logprob_config, range`。 主要通过 mock、回调或输出检查来完成验证。

### test_max_logprobs (lines 379-397)
```python
def test_max_logprobs():
    """vLLM v1 engine should fail a request with `logprobs > max_logprobs`
    Should also fail for `prompt_logprobs > max_logprobs`
    APC should not matter as this test checks basic request validation.
    """
    with VllmRunner(
        "facebook/opt-125m",
        max_logprobs=1,
        enable_prefix_caching=False,
        gpu_memory_utilization=0.15,
        max_model_len=256,
    ) as runner:
        vllm_sampling_params = SamplingParams(logprobs=1)
        # should pass
        runner.generate(["Hello world"], sampling_params=vllm_sampling_params)

        bad_sampling_params = SamplingParams(logprobs=2)
        with pytest.raises(ValueError):
            runner.generate(["Hello world"], sampling_params=bad_sampling_params)
```
**EN:** Test case covering `max logprobs`. It exercises `VllmRunner, SamplingParams, runner.generate, pytest.raises`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `max 对数概率` 的测试用例。 该测试会调用 `VllmRunner, SamplingParams, runner.generate, pytest.raises`。 主要通过预期异常检查来完成验证。

### test_none_logprobs (lines 400-425)
```python
def test_none_logprobs(vllm_model, example_prompts):
    """Engine should return `logprobs` and `prompt_logprobs` as `None`

    Args:
      vllm_model: vLLM model fixture
      example_prompts: list of example prompts (test fixture)
    """
    max_tokens = 5

    sampling_params_logprobs_none = SamplingParams(
        max_tokens=max_tokens,
        logprobs=None,
        prompt_logprobs=None,
        temperature=0.0,
    )
    results_logprobs_none = vllm_model.llm.generate(
        example_prompts,
        sampling_params=sampling_params_logprobs_none,
    )

    for i in range(len(results_logprobs_none)):
        # Check sample logprobs are None
        assert results_logprobs_none[i].outputs[0].logprobs is None
        assert results_logprobs_none[i].outputs[0].cumulative_logprob is None
        # Check prompt logprobs are None
        assert results_logprobs_none[i].prompt_logprobs is None
```
**EN:** Test case covering `none logprobs`. Inputs/fixtures: `vllm_model, example_prompts`. It exercises `SamplingParams, llm.generate, range, len`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `none 对数概率` 的测试用例。 输入或 fixture：`vllm_model, example_prompts`。 该测试会调用 `SamplingParams, llm.generate, range, len`。 代码主体包含 3 个显式断言。

### test_zero_logprobs (lines 428-457)
```python
def test_zero_logprobs(vllm_model, example_prompts):
    """Engine should return sampled token and prompt token logprobs

    Args:
      vllm_model: vLLM model fixture
      example_prompts: list of example prompts (test fixture)
    """
    max_tokens = 5

    sampling_params_logprobs_zero = SamplingParams(
        max_tokens=max_tokens, logprobs=0, prompt_logprobs=0, temperature=0.0
    )
    results_logprobs_zero = vllm_model.llm.generate(
        example_prompts, sampling_params=sampling_params_logprobs_zero
    )

    for i in range(len(results_logprobs_zero)):
        # Check that there is one sample logprob dict for each
        # sample token
        logprobs = results_logprobs_zero[i].outputs[0].logprobs
        prompt_logprobs = results_logprobs_zero[i].prompt_logprobs
        sampled_token_ids = results_logprobs_zero[i].outputs[0].token_ids
        prompt_token_ids = results_logprobs_zero[i].prompt_token_ids
        assert logprobs is not None
        assert len(sampled_token_ids) == len(logprobs)
        assert results_logprobs_zero[i].outputs[0].cumulative_logprob is not None
        # Check that there is one prompt logprob dict for each
        # prompt token
        assert prompt_logprobs is not None
        assert len(prompt_token_ids) == len(prompt_logprobs)
```
**EN:** Test case covering `zero logprobs`. Inputs/fixtures: `vllm_model, example_prompts`. It exercises `SamplingParams, llm.generate, range, len`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `zero 对数概率` 的测试用例。 输入或 fixture：`vllm_model, example_prompts`。 该测试会调用 `SamplingParams, llm.generate, range, len`。 代码主体包含 5 个显式断言。

### test_all_logprobs (lines 460-490)
```python
def test_all_logprobs(example_prompts):
    """Engine should return all vocabulary logprobs and prompt logprobs

    Args:
      example_prompts: list of example prompts (test fixture)
    """
    with VllmRunner(
        "facebook/opt-125m",
        max_logprobs=-1,
        enable_prefix_caching=False,
        gpu_memory_utilization=0.15,
        max_model_len=256,
    ) as runner:
        sampling_params_logprobs_all = SamplingParams(
            max_tokens=5, logprobs=-1, prompt_logprobs=-1
        )
        results_logprobs_all = runner.llm.generate(
            example_prompts, sampling_params=sampling_params_logprobs_all
        )
        vocab_size = runner.llm.llm_engine.model_config.get_vocab_size()

        for i in range(len(results_logprobs_all)):
            logprobs = results_logprobs_all[i].outputs[0].logprobs
            prompt_logprobs = results_logprobs_all[i].prompt_logprobs
            assert logprobs is not None
            for logprob in logprobs:
                assert len(logprob) == vocab_size
            assert prompt_logprobs is not None
            assert prompt_logprobs[0] is None
            for prompt_logprob in prompt_logprobs[1:]:
                assert len(prompt_logprob) == vocab_size
```
**EN:** Test case covering `all logprobs`. Inputs/fixtures: `example_prompts`. It exercises `VllmRunner, SamplingParams, llm.generate, model_config.get_vocab_size, range, len`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `all 对数概率` 的测试用例。 输入或 fixture：`example_prompts`。 该测试会调用 `VllmRunner, SamplingParams, llm.generate, model_config.get_vocab_size, range, len`。 代码主体包含 5 个显式断言。

### test_logprobs_mode (lines 494-531)
```python
def test_logprobs_mode(logprobs_mode: LogprobsMode):
    """Test with LLM engine with different logprobs_mode.
    For logprobs, we should have non-positive values.
    For logits, we should expect at least one positive values.
    """
    from vllm import LLM

    llm = LLM(
        "facebook/opt-125m",
        max_logprobs=5,
        enable_prefix_caching=False,
        # 2 other llms alive during whole session
        gpu_memory_utilization=0.05,
        max_model_len=16,
        logprobs_mode=logprobs_mode,
    )
    try:
        vllm_sampling_params = SamplingParams(logprobs=1)
    # ... excerpt omitted for brevity ...
                        assert logprob.logprob <= 0
        assert total_token_with_logprobs >= len(results[0].outputs)
        if logprobs_mode in ("raw_logits", "processed_logits"):
            assert positive_values > 0
    finally:
        del llm
        torch.accelerator.empty_cache()
        cleanup_dist_env_and_memory()
```
**EN:** Parameterized test covering `logprobs mode`. Parameter axes: `logprobs_mode`. Inputs/fixtures: `logprobs_mode`. It exercises `mark.parametrize, LLM, get_args, SamplingParams, llm.generate, accelerator.empty_cache`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `对数概率 mode` 的测试用例。 参数维度：`logprobs_mode`。 输入或 fixture：`logprobs_mode`。 该测试会调用 `mark.parametrize, LLM, get_args, SamplingParams, llm.generate, accelerator.empty_cache`。 代码主体包含 3 个显式断言。

### TestCorrectDecodedToken (lines 534-863)
```python
class TestCorrectDecodedToken:
    """Unit tests for _correct_decoded_token method in LogprobsProcessor.

    This method handles UTF-8 decoding issues where incomplete byte sequences
    result in the Unicode replacement character "�" (U+FFFD). This commonly
    happens with byte-fallback tokenization when multi-byte UTF-8 characters
    are split across tokens.
    The method signature is _correct_decoded_token(token_id, context_token_ids)
    where token_id is the single token to correct and context_token_ids are
    the preceding sampled tokens in sequential order.
    """
    @pytest.fixture
    def mock_tokenizer(self):
        """Create a mock tokenizer for testing."""
        from unittest.mock import Mock
    # ... excerpt omitted for brevity ...
        tokenizer = Mock()
        return tokenizer
        return processor
                return "hello valid"
                return "hello "
            return "�"
        # Token 44 is byte-fallback, so clean context is [1385].
        # decode([1385, 44, 48]) = " term \u201c"
        # decode([1385]) = " term"
        # result = " \u201c" (space preserved from token 44!)
        result = processor._correct_decoded_token(48, [1385, 44])
        assert result == " \u201c"
```
**EN:** Class `TestCorrectDecodedToken` groups 11 test method(s) and 2 helper/fixture method(s). Representative scenarios: `test_correction_with_context, test_correction_with_context_from_logprobs, test_correction_no_context, test_correction_with_context_succeeds, test_fallback_when_all_attempts_fail, test_increasing_context_window, ...`.
**CN:** 类 `TestCorrectDecodedToken` 组织了 11 个测试方法，以及 2 个辅助或 fixture 方法。 代表性场景：`test_correction_with_context, test_correction_with_context_from_logprobs, test_correction_no_context, test_correction_with_context_succeeds, test_fallback_when_all_attempts_fail, test_increasing_context_window, ...`。

### test_verify_tokens_integration (lines 866-907)
```python
def test_verify_tokens_integration():
    """Integration test for _verify_tokens with real model.

    This test validates that _verify_tokens correctly identifies and
    corrects tokens ending with the replacement character "�".
    Uses facebook/opt-125m which is known to produce these issues.
    """
    with VllmRunner(
        "facebook/opt-125m",
        max_logprobs=0,
        enable_prefix_caching=False,
        gpu_memory_utilization=0.15,
        max_model_len=256,
    ) as runner:
        # Use a prompt that triggers multi-byte UTF-8 issues
        # Based on user's example: "In this example,"
        test_prompts = ["In this example,"]
    # ... excerpt omitted for brevity ...
            assert result.outputs[0].logprobs is not None
                    assert not decoded_token.endswith("�"), (
                        f"ends with replacement character"
                    )
                    # Decoded tokens should not contain lone replacement characters
                    assert decoded_token != "�", (
                        f"Token {token_id} is a lone replacement character"
```
**EN:** Test case covering `verify tokens integration`. It exercises `VllmRunner, SamplingParams, llm.generate, logprob_dict.items, decoded_token.endswith`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `verify tokens integration` 的测试用例。 该测试会调用 `VllmRunner, SamplingParams, llm.generate, logprob_dict.items, decoded_token.endswith`。 代码主体包含 3 个显式断言。

### test_utf8_edge_cases_with_real_model (lines 910-953)
```python
def test_utf8_edge_cases_with_real_model():
    """Test various UTF-8 edge cases with a real model.

    Tests prompts that are likely to trigger byte-fallback tokenization
    and multi-byte UTF-8 splitting.
    """
    with VllmRunner(
        "facebook/opt-125m",
        max_logprobs=1,
        enable_prefix_caching=False,
        gpu_memory_utilization=0.15,
        max_model_len=256,
    ) as runner:
        # Prompts with various multi-byte UTF-8 characters
        test_prompts = [
            'Smart quotes: "Hello"',  # Curly quotes
            "Em dash — test",  # Em dash
            "Ellipsis… continues",  # Ellipsis
    # ... excerpt omitted for brevity ...
            assert result.outputs[0].logprobs is not None
                    decoded_token = logprob_info.decoded_token
                    assert not decoded_token.endswith("�"), (
                        f"Prompt: '{prompt}'\n"
                        f"Token {token_id} decoded to '{decoded_token}' which "
                        f"ends with replacement character"
                    )
```
**EN:** Test case covering `utf8 edge cases with real model`. It exercises `VllmRunner, SamplingParams, llm.generate, enumerate, logprob_dict.items, decoded_token.endswith`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `utf8 edge cases with real model` 的测试用例。 该测试会调用 `VllmRunner, SamplingParams, llm.generate, enumerate, logprob_dict.items, decoded_token.endswith`。 代码主体包含 2 个显式断言。

### test_correct_decoded_token_preserves_valid_tokens (lines 956-991)
```python
def test_correct_decoded_token_preserves_valid_tokens():
    """Test that valid tokens (not ending with �) are not modified.

    The _correct_decoded_token method should only be called for tokens
    ending with "�", but this test verifies the broader _verify_tokens
    logic doesn't affect valid tokens.
    """
    with VllmRunner(
        "facebook/opt-125m",
        max_logprobs=2,
        enable_prefix_caching=False,
        gpu_memory_utilization=0.15,
        max_model_len=256,
    ) as runner:
        # Simple prompt with standard ASCII characters
        test_prompts = ["Hello world, this is a test."]
        sampling_params = SamplingParams(
    # ... excerpt omitted for brevity ...
            assert result.outputs[0].logprobs is not None
                for token_id, logprob_info in logprob_dict.items():
                    decoded_token = logprob_info.decoded_token
                    # Valid tokens should be non-empty strings (or empty if corrected)
                    assert isinstance(decoded_token, str)
                    # Should not contain replacement character
                    assert "�" not in decoded_token
```
**EN:** Test case covering `correct decoded token preserves valid tokens`. It exercises `VllmRunner, SamplingParams, llm.generate, logprob_dict.items, isinstance`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `correct decoded token preserves valid tokens` 的测试用例。 该测试会调用 `VllmRunner, SamplingParams, llm.generate, logprob_dict.items, isinstance`。 代码主体包含 3 个显式断言。

### test_spec_decode_logprobs (lines 1043-1157)
```python
def test_spec_decode_logprobs(
    logprobs_mode: LogprobsMode,
    model_setup: tuple[str, str, dict, int],
    monkeypatch,
):
    """Spec decode logprobs should match those of the base model.

    Runs the base model and spec decode model sequentially, ensuring
    only one LLM instance is alive at a time to avoid GPU memory
    contention. Both use identical chunked prefill settings and eager
    mode to control for infrastructure differences.
    Args:
        logprobs_mode: logprobs mode.
        model_setup: Tuple of (method, base model name,
            speculative_config dict, top_logprobs).
        monkeypatch: pytest fixture for setting env vars.
    """
    # ... excerpt omitted for brevity ...
    # Per-token logprobs are expected to be the same.
    assert len(ref_logprobs) == len(spec_logprobs)
        assert math.isclose(
        assert ref_logprob.rank == spec_logprob.rank, (
            f"Rank mismatch: ref={ref_logprob.rank} "
            f"spec={spec_logprob.rank} "
            f"(token={ref_logprob.decoded_token!r})"
        )
        assert ref_logprob.decoded_token == spec_logprob.decoded_token
```
**EN:** Parameterized test covering `spec decode logprobs`. Parameter axes: `logprobs_mode, model_setup`. Inputs/fixtures: `logprobs_mode, model_setup, monkeypatch`. It exercises `mark.parametrize, monkeypatch.setenv, SamplingParams, dict, LLM, ref_llm.generate`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `推测解码 对数概率` 的测试用例。 参数维度：`logprobs_mode, model_setup`。 输入或 fixture：`logprobs_mode, model_setup, monkeypatch`。 该测试会调用 `mark.parametrize, monkeypatch.setenv, SamplingParams, dict, LLM, ref_llm.generate`。 代码主体包含 4 个显式断言。

### test_prompt_logprobs_with_chunking_and_preemption (lines 1160-1233)
```python
def test_prompt_logprobs_with_chunking_and_preemption():
    """Test that prompt logprobs are correctly returned when using
    both chunked prefill and preemption.

    This test ensures that the num_prompt_logprobs tracking persists
    across preemptions and prefill chunks.
    """
    # Create prompts that will trigger chunking and preemption
    prompts = [
        "The following numbers of the sequence "
        + ", ".join(str(i) for i in range(10))
        + " are:",
        "In one word, the capital of France is ",
    ] + [f"Tell me about the number {i}: " for i in range(32)]
    sampling_params = SamplingParams(
        temperature=0.0,
    # ... excerpt omitted for brevity ...
            assert prompt_logprobs is not None and len(prompt_logprobs) > 0, (
            assert len(prompt_logprobs) == len(prompt_token_ids), (
                "Unexpected number of prompt logprob positions"
                    assert (
                        f"logprobs, expected {sampling_params.prompt_logprobs}"
            (m.value for m in metrics_after if m.name == "vllm:num_preemptions"), 0
        )
        preemptions = preemptions_after - preemptions_before
        assert preemptions > 0, "Test did not trigger any preemptions"
        print(f"Test passed with {preemptions} preemptions")
```
**EN:** Test case covering `prompt logprobs with chunking and preemption`. It exercises `SamplingParams, VllmRunner, llm.get_metrics, vllm_model.generate_w_logprobs, enumerate, next`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt 对数概率 with chunking and preemption` 的测试用例。 该测试会调用 `SamplingParams, VllmRunner, llm.get_metrics, vllm_model.generate_w_logprobs, enumerate, next`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm, vllm.config.model, vllm.distributed, vllm.platforms, vllm.v1.engine.logprobs`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config.model, vllm.distributed, vllm.platforms, vllm.v1.engine.logprobs`。
- **EN:** Local test helpers: `tests.utils, tests.v1.sample.utils, tests.conftest`.
- **CN:** 本地测试辅助模块：`tests.utils, tests.v1.sample.utils, tests.conftest`。
- **EN:** Standard-library support: `itertools, math, collections.abc, typing, unittest.mock`.
- **CN:** 标准库支持：`itertools, math, collections.abc, typing, unittest.mock`。
