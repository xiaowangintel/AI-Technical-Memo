# test_spec_decode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/e2e/spec_decode/test_spec_decode.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises end-to-end `spec decode` scenarios in the v1 test suite. / 在 v1 测试套件中覆盖端到端的 `推测解码` 场景。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-29)
```python
import os
import random
from collections.abc import Iterable
from dataclasses import dataclass
from typing import Any

import pytest
import torch
from tqdm import tqdm

from tests.evals.gsm8k.gsm8k_eval import _build_gsm8k_prompts, evaluate_gsm8k_offline
from tests.utils import (
    get_attn_backend_list_based_on_platform,
    large_gpu_mark,
    multi_gpu_marks,
    multi_gpu_only,
    single_gpu_only,
)
from vllm import LLM, SamplingParams
from vllm.assets.base import VLLM_S3_BUCKET_URL
from vllm.assets.image import VLM_IMAGES_DIR
from vllm.benchmarks.datasets import InstructCoderDataset
from vllm.config import VllmConfig, replace
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.engine.arg_utils import EngineArgs
from vllm.platforms import current_platform
from vllm.v1.metrics.reader import Metric
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, tqdm, datasets, transformers, ...`. vLLM modules under test include `vllm, vllm.assets.base, vllm.assets.image, vllm.benchmarks.datasets, vllm.config, ...`. Local helpers come from `tests.evals.gsm8k.gsm8k_eval, tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, tqdm, datasets, transformers, ...`。 被测试的 vLLM 模块包括 `vllm, vllm.assets.base, vllm.assets.image, vllm.benchmarks.datasets, vllm.config, ...`。 本地测试辅助逻辑来自 `tests.evals.gsm8k.gsm8k_eval, tests.utils`。

### Module state / 模块级状态 (line 31)
```python
MTP_SIMILARITY_RATE = 0.8
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MTP_SIMILARITY_RATE`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MTP_SIMILARITY_RATE`。

### AsyncSchedulingNotEnabledError (lines 34-38)
```python
class AsyncSchedulingNotEnabledError(AssertionError):
    """Raised when async_scheduling is expected to be True for draft_model
    spec decode but is False. Tracked in:
    https://github.com/vllm-project/vllm/issues/38929
    """
```
**EN:** Class `AsyncSchedulingNotEnabledError` groups 0 test method(s). Bases: `AssertionError`.
**CN:** 类 `AsyncSchedulingNotEnabledError` 组织了 0 个测试方法。 基类：`AssertionError`。

### _skip_if_insufficient_gpus_for_tp (lines 41-47)
```python
def _skip_if_insufficient_gpus_for_tp(tp_size: int):
    """Skip test if available GPUs < tp_size on ROCm."""
    available_gpus = torch.accelerator.device_count()
    if available_gpus < tp_size:
        pytest.skip(
            f"Test requires {tp_size} GPUs, but only {available_gpus} available"
        )
```
**EN:** Helper function `_skip_if_insufficient_gpus_for_tp` encapsulates reusable logic for `skip if insufficient gpus for tp`. Inputs: `tp_size`. Key calls include `accelerator.device_count, pytest.skip`.
**CN:** 辅助函数 `_skip_if_insufficient_gpus_for_tp` 封装了与 `skip if insufficient gpus for tp` 相关的可复用逻辑。 输入参数：`tp_size`。 关键调用包括 `accelerator.device_count, pytest.skip`。

### Module state / 模块级状态 (line 50)
```python
Messages = list[dict[str, Any]]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `Messages`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`Messages`。

### get_test_prompts (lines 53-106)
```python
def get_test_prompts(mm_enabled: bool, num_prompts: int = 100) -> list[Messages]:
    prompt_types = ["repeat", "gsm8k"]
    if mm_enabled:
        prompt_types.append("mm")
    prompts: list[Messages] = []

    num_repeat_prompts = num_prompts // len(prompt_types)
        num_gsm8k_prompts = num_prompts // len(prompt_types)
        num_mm_prompts = num_prompts - num_repeat_prompts - num_gsm8k_prompts
    else:
        num_mm_prompts = 0
        num_gsm8k_prompts = num_prompts - num_repeat_prompts
    # Generate a mixed batch of prompts, some of which can be easily
    # predicted by n-gram matching and some which likely cannot.
    random.seed(0)
    for _ in range(num_repeat_prompts):
    # ... excerpt omitted for brevity ...
            *placeholders,
            {"type": "text", "text": "The meaning of the image is"},
        ]
        prompts.append([{"role": "user", "content": prompt}])
    return prompts
```
**EN:** Helper function `get_test_prompts` encapsulates reusable logic for `test prompts`. Inputs: `mm_enabled, num_prompts`. Key calls include `random.seed, range, prompts.extend, prompt_types.append, len, random.choice`.
**CN:** 辅助函数 `get_test_prompts` 封装了与 `test prompts` 相关的可复用逻辑。 输入参数：`mm_enabled, num_prompts`。 关键调用包括 `random.seed, range, prompts.extend, prompt_types.append, len, random.choice`。

### get_instruct_coder_messages (lines 109-114)
```python
def get_instruct_coder_messages(n: int) -> list[Messages]:
    dataset = InstructCoderDataset(
        dataset_path="likaixin/InstructCoder", dataset_split="train"
    )
    prompts: Iterable[str] = dataset.sample_prompts(n=n)
    return [[{"role": "user", "content": prompt}] for prompt in prompts]
```
**EN:** Helper function `get_instruct_coder_messages` encapsulates reusable logic for `instruct coder messages`. Inputs: `n`. Key calls include `InstructCoderDataset, dataset.sample_prompts`.
**CN:** 辅助函数 `get_instruct_coder_messages` 封装了与 `instruct coder messages` 相关的可复用逻辑。 输入参数：`n`。 关键调用包括 `InstructCoderDataset, dataset.sample_prompts`。

### sampling_config (lines 118-119)
```python
def sampling_config():
    return greedy_sampling()
```
**EN:** Fixture/helper `sampling_config` prepares reusable state for downstream tests. Key calls include `greedy_sampling`.
**CN:** `sampling_config` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `greedy_sampling`。

### greedy_sampling (lines 122-123)
```python
def greedy_sampling() -> SamplingParams:
    return SamplingParams(temperature=0, max_tokens=10, ignore_eos=False)
```
**EN:** Helper function `greedy_sampling` encapsulates reusable logic for `greedy sampling`. Key calls include `SamplingParams`.
**CN:** 辅助函数 `greedy_sampling` 封装了与 `greedy 采样` 相关的可复用逻辑。 关键调用包括 `SamplingParams`。

### stochastic_sampling (lines 126-127)
```python
def stochastic_sampling() -> SamplingParams:
    return SamplingParams(temperature=1.0, max_tokens=10, ignore_eos=False)
```
**EN:** Helper function `stochastic_sampling` encapsulates reusable logic for `stochastic sampling`. Key calls include `SamplingParams`.
**CN:** 辅助函数 `stochastic_sampling` 封装了与 `stochastic 采样` 相关的可复用逻辑。 关键调用包括 `SamplingParams`。

### model_name (lines 131-132)
```python
def model_name():
    return "meta-llama/Llama-3.1-8B-Instruct"
```
**EN:** Fixture/helper `model_name` prepares reusable state for downstream tests.
**CN:** `model_name` 是为后续测试准备可复用状态的 fixture/辅助函数。

### evaluate_llm_for_gsm8k (lines 135-151)
```python
def evaluate_llm_for_gsm8k(llm: LLM, expected_accuracy_threshold: float = 0.70) -> None:
    """Evaluate the LLM on GSM8K and check that accuracy is above a sanity threshold.

    The default threshold assumes the LLM uses the same target model as the "model_name"
    fixture, with max model len == 4096. Precomputed reference value is 75% to 80%
    on GSM8K with greedy decoding, so we check that it's above a sanity threshold of 70%
    to verify that the model is correct.
    """
    if expected_accuracy_threshold <= 0.0:
        print("Skipping GSM8K evaluation")
        return
    results = evaluate_gsm8k_offline(llm)
    accuracy = results["accuracy"]
    print(f"GSM8K accuracy: {accuracy:.3f}")
    assert accuracy >= expected_accuracy_threshold, (
        f"Expected GSM8K accuracy >= {expected_accuracy_threshold}, got {accuracy:.3f}"
    )
```
**EN:** Helper function `evaluate_llm_for_gsm8k` encapsulates reusable logic for `evaluate LLM for gsm8k`. Inputs: `llm, expected_accuracy_threshold`. Key calls include `evaluate_gsm8k_offline, print`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `evaluate_llm_for_gsm8k` 封装了与 `evaluate llm for gsm8k` 相关的可复用逻辑。 输入参数：`llm, expected_accuracy_threshold`。 关键调用包括 `evaluate_gsm8k_offline, print`。 其中包含 1 个内部断言，用于保护前置假设。

### reset_torch_dynamo (lines 155-159)
```python
def reset_torch_dynamo():
    """Reset torch dynamo cache before each test"""
    yield
    # Cleanup after test
    torch._dynamo.reset()
```
**EN:** Fixture/helper `reset_torch_dynamo` prepares reusable state for downstream tests. Key calls include `pytest.fixture, _dynamo.reset`.
**CN:** `reset_torch_dynamo` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `pytest.fixture, _dynamo.reset`。

### test_ngram_and_suffix_correctness (lines 179-191)
```python
def test_ngram_and_suffix_correctness(
    speculative_config: dict,
    model_name: str,
):
    spec_llm = LLM(
        model=model_name,
        speculative_config=speculative_config,
        max_model_len=4096,
    )
    evaluate_llm_for_gsm8k(spec_llm)
    del spec_llm
    torch.accelerator.empty_cache()
    cleanup_dist_env_and_memory()
```
**EN:** Parameterized test covering `ngram and suffix correctness`. Parameter axes: `speculative_config`. Inputs/fixtures: `speculative_config, model_name`. It exercises `mark.parametrize, large_gpu_mark, LLM, evaluate_llm_for_gsm8k, accelerator.empty_cache, cleanup_dist_env_and_memory`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `ngram and suffix correctness` 的测试用例。 参数维度：`speculative_config`。 输入或 fixture：`speculative_config, model_name`。 该测试会调用 `mark.parametrize, large_gpu_mark, LLM, evaluate_llm_for_gsm8k, accelerator.empty_cache, cleanup_dist_env_and_memory`。 主要通过 mock、回调或输出检查来完成验证。

### test_ngram_gpu_default_with_async_scheduling (lines 197-224)
```python
def test_ngram_gpu_default_with_async_scheduling(
    async_scheduling: bool,
):
    """
    Test ngram_gpu speculative decoding (k=3) correctness with and without
    async scheduling, validated via GSM8K accuracy.
    Uses Qwen/Qwen3-8B (ref GSM8K accuracy: 87%-92%).
    """
    qwen3_model = "Qwen/Qwen3-8B"
    spec_llm = LLM(
        model=qwen3_model,
        speculative_config={
            "method": "ngram_gpu",
            "prompt_lookup_max": 3,
            "prompt_lookup_min": 2,
            "num_speculative_tokens": 2,
        },
        max_model_len=4096,
        async_scheduling=async_scheduling,
    )
    # Assert the resolved async_scheduling config matches what was requested.
    assert (
        spec_llm.llm_engine.vllm_config.scheduler_config.async_scheduling
        == async_scheduling
    )
    evaluate_llm_for_gsm8k(spec_llm, expected_accuracy_threshold=0.8)
    del spec_llm
    cleanup_dist_env_and_memory()
```
**EN:** Parameterized test covering `ngram GPU default with async scheduling`. Parameter axes: `async_scheduling`. Inputs/fixtures: `async_scheduling`. It exercises `mark.parametrize, large_gpu_mark, LLM, evaluate_llm_for_gsm8k, cleanup_dist_env_and_memory`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `ngram gpu default with async scheduling` 的测试用例。 参数维度：`async_scheduling`。 输入或 fixture：`async_scheduling`。 该测试会调用 `mark.parametrize, large_gpu_mark, LLM, evaluate_llm_for_gsm8k, cleanup_dist_env_and_memory`。 代码主体包含 1 个显式断言。

### test_suffix_decoding_acceptance (lines 229-285)
```python
def test_suffix_decoding_acceptance(
    monkeypatch: pytest.MonkeyPatch,
    sampling_config: SamplingParams,
    model_name: str,
):
    """
    Check that suffix decoding caching takes effect and improves acceptance
    lengths and acceptance rates over multiple runs of the same prompts.
    test_prompts = get_test_prompts(mm_enabled=False)

    spec_llm = LLM(
        model=model_name,
        speculative_config={
            "method": "suffix",
            "suffix_decoding_max_spec_factor": 2.0,
            "suffix_decoding_max_cached_requests": 1000,
        },
    # ... excerpt omitted for brevity ...
    assert first_accept_tokens < last_accept_tokens
    assert first_accept_rate < last_accept_rate
    # Heuristic: expect at least 80.0% acceptance rate at the end.
    assert last_accept_rate > 0.80
    del spec_llm
    torch.accelerator.empty_cache()
    cleanup_dist_env_and_memory()
```
**EN:** Test case covering `suffix decoding acceptance`. Inputs/fixtures: `monkeypatch, sampling_config, model_name`. It exercises `large_gpu_mark, get_test_prompts, LLM, range, accelerator.empty_cache, cleanup_dist_env_and_memory`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `suffix decoding acceptance` 的测试用例。 输入或 fixture：`monkeypatch, sampling_config, model_name`。 该测试会调用 `large_gpu_mark, get_test_prompts, LLM, range, accelerator.empty_cache, cleanup_dist_env_and_memory`。 代码主体包含 3 个显式断言。

### test_speculators_model_integration (lines 298-372)
```python
def test_speculators_model_integration(
    monkeypatch: pytest.MonkeyPatch,
    sampling_config: SamplingParams,
    model_path: str,
    expected_accuracy_threshold: float,
):
    """
    Test that speculators models work with the simplified integration.

    This verifies the `vllm serve <speculator-model>` use case where
    speculative config is automatically detected from the model config
    without requiring explicit --speculative-config argument.
    Tests:
    1. Speculator model is correctly detected
    2. Verifier model is extracted from speculator config
    3. Speculative decoding is automatically enabled
    4. Text generation works correctly
    # ... excerpt omitted for brevity ...
        spec_llm, expected_accuracy_threshold=expected_accuracy_threshold
    assert spec_llm.llm_engine.vllm_config.speculative_config is not None, (
    assert spec_config.num_speculative_tokens > 0, (
    assert spec_config.model == model_path, (
    # Heuristic: expect at least 66% of prompts to match exactly
    assert matches >= int(0.66 * len(ref_outputs)), (
        f"Only {matches}/{len(ref_outputs)} outputs matched. "
        f"Expected at least {int(0.66 * len(ref_outputs))} matches."
    )
```
**EN:** Test case covering `speculators model integration`. Inputs/fixtures: `monkeypatch, sampling_config, model_path, expected_accuracy_threshold`. It exercises `mark.parametrize, large_gpu_mark, monkeypatch.setenv, get_test_prompts, LLM, evaluate_llm_for_gsm8k`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `speculators model integration` 的测试用例。 输入或 fixture：`monkeypatch, sampling_config, model_path, expected_accuracy_threshold`。 该测试会调用 `mark.parametrize, large_gpu_mark, monkeypatch.setenv, get_test_prompts, LLM, evaluate_llm_for_gsm8k`。 代码主体包含 4 个显式断言。

### _run_eagle_correctness (lines 375-487)
```python
def _run_eagle_correctness(
    monkeypatch: pytest.MonkeyPatch,
    sampling_config: SamplingParams,
    model_setup: tuple[str, str, str, int],
    mm_enabled: bool,
    expected_accuracy_threshold: float,
    enable_chunked_prefill: bool,
    model_impl: str,
    attn_backend: str,
):
    """
    Compare the outputs of an original LLM and a speculative LLM
    which should be the same when using eagle speculative decoding.
    if model_impl == "transformers":
        import transformers
        from packaging.version import Version

    # ... excerpt omitted for brevity ...
            ref_llm, expected_accuracy_threshold=expected_accuracy_threshold
        assert spec_llm.llm_engine.vllm_config.scheduler_config.async_scheduling
            spec_llm, expected_accuracy_threshold=expected_accuracy_threshold
                print(f"spec_output: {spec_output.outputs[0].text}")
        assert matches > int(0.6 * len(ref_outputs))
        del spec_llm
        torch.accelerator.empty_cache()
        cleanup_dist_env_and_memory()
```
**EN:** Helper function `_run_eagle_correctness` encapsulates reusable logic for `run eagle correctness`. Inputs: `monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`. Key calls include `get_test_prompts, Version, current_platform.is_rocm, pytest.skip, monkeypatch.context, m.setenv`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_run_eagle_correctness` 封装了与 `run eagle correctness` 相关的可复用逻辑。 输入参数：`monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`。 关键调用包括 `get_test_prompts, Version, current_platform.is_rocm, pytest.skip, monkeypatch.context, m.setenv`。 其中包含 2 个内部断言，用于保护前置假设。

### test_eagle_correctness_light (lines 520-539)
```python
def test_eagle_correctness_light(
    monkeypatch: pytest.MonkeyPatch,
    sampling_config: SamplingParams,
    model_setup: tuple[str, str, str, int],
    mm_enabled: bool,
    expected_accuracy_threshold: float,
    enable_chunked_prefill: bool,
    model_impl: str,
    attn_backend: str,
):
    _run_eagle_correctness(
        monkeypatch,
        sampling_config,
        model_setup,
        mm_enabled,
        expected_accuracy_threshold,
        enable_chunked_prefill,
        model_impl,
        attn_backend,
    )
```
**EN:** Parameterized test covering `eagle correctness light`. Parameter axes: `attn_backend`. Inputs/fixtures: `monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`. It exercises `mark.skipif, mark.parametrize, _run_eagle_correctness, current_platform.is_device_capability_family, get_attn_backend_list_based_on_platform`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `eagle correctness light` 的测试用例。 参数维度：`attn_backend`。 输入或 fixture：`monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`。 该测试会调用 `mark.skipif, mark.parametrize, _run_eagle_correctness, current_platform.is_device_capability_family, get_attn_backend_list_based_on_platform`。 主要通过 mock、回调或输出检查来完成验证。

### test_eagle_correctness_medium (lines 623-642)
```python
def test_eagle_correctness_medium(
    monkeypatch: pytest.MonkeyPatch,
    sampling_config: SamplingParams,
    model_setup: tuple[str, str, str, int],
    mm_enabled: bool,
    expected_accuracy_threshold: float,
    enable_chunked_prefill: bool,
    model_impl: str,
    attn_backend: str,
):
    _run_eagle_correctness(
        monkeypatch,
        sampling_config,
        model_setup,
        mm_enabled,
        expected_accuracy_threshold,
        enable_chunked_prefill,
        model_impl,
        attn_backend,
    )
```
**EN:** Parameterized test covering `eagle correctness medium`. Parameter axes: `attn_backend`. Inputs/fixtures: `monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`. It exercises `large_gpu_mark, mark.parametrize, _run_eagle_correctness, get_attn_backend_list_based_on_platform, pytest.param, mark.skip`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `eagle correctness medium` 的测试用例。 参数维度：`attn_backend`。 输入或 fixture：`monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`。 该测试会调用 `large_gpu_mark, mark.parametrize, _run_eagle_correctness, get_attn_backend_list_based_on_platform, pytest.param, mark.skip`。 主要通过 mock、回调或输出检查来完成验证。

### test_eagle_correctness_heavy (lines 699-718)
```python
def test_eagle_correctness_heavy(
    monkeypatch: pytest.MonkeyPatch,
    sampling_config: SamplingParams,
    model_setup: tuple[str, str, str, int],
    mm_enabled: bool,
    expected_accuracy_threshold: float,
    enable_chunked_prefill: bool,
    model_impl: str,
    attn_backend: str,
):
    _run_eagle_correctness(
        monkeypatch,
        sampling_config,
        model_setup,
        mm_enabled,
        expected_accuracy_threshold,
        enable_chunked_prefill,
        model_impl,
        attn_backend,
    )
```
**EN:** Parameterized test covering `eagle correctness heavy`. Parameter axes: `attn_backend`. Inputs/fixtures: `monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`. It exercises `mark.parametrize, _run_eagle_correctness, get_attn_backend_list_based_on_platform, pytest.param, large_gpu_mark, multi_gpu_marks`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `eagle correctness heavy` 的测试用例。 参数维度：`attn_backend`。 输入或 fixture：`monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold, enable_chunked_prefill, model_impl, attn_backend`。 该测试会调用 `mark.parametrize, _run_eagle_correctness, get_attn_backend_list_based_on_platform, pytest.param, large_gpu_mark, multi_gpu_marks`。 主要通过 mock、回调或输出检查来完成验证。

### test_mtp_correctness (lines 749-855)
```python
def test_mtp_correctness(
    monkeypatch: pytest.MonkeyPatch,
    sampling_config: SamplingParams,
    model_setup: tuple[str, str, int] | tuple[str, str, int, str],
    mm_enabled: bool,
    expected_accuracy_threshold: float,
):
    """
    Compare the outputs of a original LLM and a speculative LLM
    which should be the same when using MTP speculative decoding. Due to some variance
    in the engine, it is possible for some outputs to differ, so we expect that at least
    6/10 output tokens match exactly, and that the GSM8k accuracy is above a precomputed
    reference threshold for each model.
    # Generate test prompts inside the function instead of using fixture
    test_prompts = get_test_prompts(mm_enabled)
    with monkeypatch.context() as m:
        m.setenv("VLLM_MLA_DISABLE", "1")
    # ... excerpt omitted for brevity ...
            ref_llm, expected_accuracy_threshold=expected_accuracy_threshold
        assert spec_llm.llm_engine.vllm_config.scheduler_config.async_scheduling
            spec_llm, expected_accuracy_threshold=expected_accuracy_threshold
        # Heuristic: expect at least 80% of the prompts to match exactly
        # Upon failure, inspect the outputs to check for inaccuracy.
        assert matches > int(MTP_SIMILARITY_RATE * len(ref_outputs))
        del spec_llm
        torch.accelerator.empty_cache()
        cleanup_dist_env_and_memory()
```
**EN:** Test case covering `mtp correctness`. Inputs/fixtures: `monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold`. It exercises `mark.parametrize, large_gpu_mark, get_test_prompts, monkeypatch.context, m.setenv, _skip_if_insufficient_gpus_for_tp`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `mtp correctness` 的测试用例。 输入或 fixture：`monkeypatch, sampling_config, model_setup, mm_enabled, expected_accuracy_threshold`。 该测试会调用 `mark.parametrize, large_gpu_mark, get_test_prompts, monkeypatch.context, m.setenv, _skip_if_insufficient_gpus_for_tp`。 代码主体包含 2 个显式断言。

### ArgsTest (lines 859-875)
```python
class ArgsTest:
    target_model: str
    draft_model: str
    sampling_config: SamplingParams
    num_speculative_tokens: int
    expected_acceptance_rate: float
    expected_acceptance_len: float
    expected_gsm8k_accuracy: float = 0.0  # skip by default
    # Defaults
    enforce_eager: bool = True
    parallel_drafting: bool = False
    target_tensor_parallel_size: int = 1
    draft_tensor_parallel_size: int = 1
    max_model_len: int = 2048
    gpu_memory_utilization: float = 0.5
    dataset: str = "test_prompts"
    num_prompts: int = 100
```
**EN:** Class `ArgsTest` groups 0 test method(s).
**CN:** 类 `ArgsTest` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 878-899)
```python
cases = [
    # Same model for draft and target, greedy sampling.
    ArgsTest(
        target_model="Qwen/Qwen3-0.6B",
        draft_model="Qwen/Qwen3-0.6B",
        sampling_config=greedy_sampling(),
        num_speculative_tokens=3,  # K
        expected_acceptance_len=0.98 * (3 + 1),  # epsilon discount of K + 1
        expected_acceptance_rate=0.98,  # slight epsilon
        expected_gsm8k_accuracy=0.25,  # ref: 35-40%
    ),
    # Smaller draft model, stochastic sampling.
    ArgsTest(
        target_model="Qwen/Qwen3-1.7B",
        draft_model="Qwen/Qwen3-0.6B",
        sampling_config=stochastic_sampling(),
        num_speculative_tokens=3,
        expected_acceptance_len=3.4,  # ref: 3.7
        expected_acceptance_rate=0.80,  # ref: 0.90
        expected_gsm8k_accuracy=0.5,  # ref: 60%. Note gsm8k always runs greedy sampling
    ),
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `cases`. Shared setup calls include `ArgsTest, greedy_sampling, stochastic_sampling`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`cases`。 共享初始化调用包括 `ArgsTest, greedy_sampling, stochastic_sampling`。

### test_draft_model_correctness (lines 910-912)
```python
def test_draft_model_correctness(args: ArgsTest, enforce_eager: bool):
    args.enforce_eager = enforce_eager
    assert_draft_model_correctness(args)
```
**EN:** Parameterized test covering `draft model correctness`. Parameter axes: `args, enforce_eager`. Inputs/fixtures: `args, enforce_eager`. It exercises `mark.parametrize, mark.xfail, assert_draft_model_correctness`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `draft model correctness` 的测试用例。 参数维度：`args, enforce_eager`。 输入或 fixture：`args, enforce_eager`。 该测试会调用 `mark.parametrize, mark.xfail, assert_draft_model_correctness`。 主要通过 mock、回调或输出检查来完成验证。

### test_draft_model_realistic_example (lines 921-932)
```python
def test_draft_model_realistic_example():
    args = ArgsTest(
        target_model="Qwen/Qwen3-1.7B",
        draft_model="Qwen/Qwen3-0.6B",
        dataset="likaixin/InstructCoder",
        num_speculative_tokens=3,
        sampling_config=greedy_sampling(),
        enforce_eager=False,
        expected_acceptance_len=2.6,  # ref: 2.86
        expected_acceptance_rate=0.5,  # ref: 0.62
    )
    assert_draft_model_correctness(args)
```
**EN:** Test case covering `draft model realistic example`. It exercises `mark.xfail, ArgsTest, assert_draft_model_correctness, greedy_sampling`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `draft model realistic example` 的测试用例。 该测试会调用 `mark.xfail, ArgsTest, assert_draft_model_correctness, greedy_sampling`。 主要通过 mock、回调或输出检查来完成验证。

### test_draft_model_parallel_drafting (lines 941-953)
```python
def test_draft_model_parallel_drafting():
    args = ArgsTest(
        target_model="Qwen/Qwen3-1.7B",
        draft_model="amd/PARD-Qwen3-0.6B",
        dataset="likaixin/InstructCoder",
        num_speculative_tokens=3,
        sampling_config=greedy_sampling(),
        parallel_drafting=True,
        enforce_eager=False,
        expected_acceptance_len=2.3,  # ref: 2.52
        expected_acceptance_rate=0.4,  # ref: 0.51
    )
    assert_draft_model_correctness(args)
```
**EN:** Test case covering `draft model parallel drafting`. It exercises `mark.xfail, ArgsTest, assert_draft_model_correctness, greedy_sampling`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `draft model parallel drafting` 的测试用例。 该测试会调用 `mark.xfail, ArgsTest, assert_draft_model_correctness, greedy_sampling`。 主要通过 mock、回调或输出检查来完成验证。

### test_draft_model_quantization (lines 972-980)
```python
def test_draft_model_quantization(models: tuple[str, str], enforce_eager: bool):
    tgt_model, draft_model = models
    sd_case = ArgsTest(
        target_model=tgt_model,
        draft_model=draft_model,
        **some_high_acceptance_metrics(),
        enforce_eager=enforce_eager,
    )
    assert_draft_model_correctness(sd_case)
```
**EN:** Parameterized test covering `draft model quantization`. Parameter axes: `models, enforce_eager`. Inputs/fixtures: `models, enforce_eager`. It exercises `mark.parametrize, mark.xfail, ArgsTest, assert_draft_model_correctness, some_high_acceptance_metrics`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `draft model quantization` 的测试用例。 参数维度：`models, enforce_eager`。 输入或 fixture：`models, enforce_eager`。 该测试会调用 `mark.parametrize, mark.xfail, ArgsTest, assert_draft_model_correctness, some_high_acceptance_metrics`。 主要通过 mock、回调或输出检查来完成验证。

### test_draft_model_tensor_parallelism (lines 989-1001)
```python
def test_draft_model_tensor_parallelism():
    """Ensure spec decode works when running with TP > 1."""
    _skip_if_insufficient_gpus_for_tp(2)
    sd_case = ArgsTest(
        target_model="Qwen/Qwen3-1.7B",
        target_tensor_parallel_size=2,
        draft_model="Qwen/Qwen3-0.6B",
        draft_tensor_parallel_size=2,
        **some_high_acceptance_metrics(),
        enforce_eager=False,
        expected_gsm8k_accuracy=0.5,
    )
    assert_draft_model_correctness(sd_case)
```
**EN:** Test case covering `draft model tensor parallelism`. It exercises `multi_gpu_only, mark.xfail, _skip_if_insufficient_gpus_for_tp, ArgsTest, assert_draft_model_correctness, some_high_acceptance_metrics`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `draft model tensor parallelism` 的测试用例。 该测试会调用 `multi_gpu_only, mark.xfail, _skip_if_insufficient_gpus_for_tp, ArgsTest, assert_draft_model_correctness, some_high_acceptance_metrics`。 主要通过 mock、回调或输出检查来完成验证。

### test_draft_model_engine_args_tensor_parallelism (lines 1005-1035)
```python
def test_draft_model_engine_args_tensor_parallelism():
    """Ensure the vllm_config for the draft model is created correctly,
    and independently of the target model (quantization, TP, etc.)"""
    _skip_if_insufficient_gpus_for_tp(2)

    engine_args = EngineArgs(
        model="Qwen/Qwen3-1.7B-FP8",  # <<< tgt quantized
        tensor_parallel_size=2,
        speculative_config={
            "model": "Qwen/Qwen3-0.6B",  # <<< draft not quantized
            "method": "draft_model",
            "num_speculative_tokens": 3,
            "draft_tensor_parallel_size": 1,  # <<< valid arg name
        },
    )
    target_config: VllmConfig = engine_args.create_engine_config()
    assert target_config.parallel_config.tensor_parallel_size == 2
    assert target_config.quant_config.get_name() == "fp8"

    speculative_config = target_config.speculative_config
    draft_config: VllmConfig = replace(
        target_config,
        quant_config=None,
        parallel_config=replace(
            speculative_config.draft_parallel_config,
            rank=target_config.parallel_config.rank,
        ),
        model_config=speculative_config.draft_model_config,
    )
    assert draft_config.parallel_config.tensor_parallel_size == 1
    assert draft_config.quant_config is None
```
**EN:** Test case covering `draft model engine args tensor parallelism`. It exercises `multi_gpu_only, _skip_if_insufficient_gpus_for_tp, EngineArgs, engine_args.create_engine_config, replace, quant_config.get_name`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `draft model 引擎 args tensor parallelism` 的测试用例。 该测试会调用 `multi_gpu_only, _skip_if_insufficient_gpus_for_tp, EngineArgs, engine_args.create_engine_config, replace, quant_config.get_name`。 代码主体包含 4 个显式断言。

### _apply_draft_moe_backend (lines 1038-1050)
```python
def _apply_draft_moe_backend(vllm_config: VllmConfig) -> VllmConfig:
    """Replicate SpecDecodeBaseProposer._create_draft_vllm_config logic
    so we can test it without instantiating a full proposer."""
    spec_cfg = vllm_config.speculative_config
    if spec_cfg.moe_backend is not None:
        return replace(
            vllm_config,
            kernel_config=replace(
                vllm_config.kernel_config,
                moe_backend=spec_cfg.moe_backend,
            ),
        )
    return vllm_config
```
**EN:** Helper function `_apply_draft_moe_backend` encapsulates reusable logic for `apply draft moe backend`. Inputs: `vllm_config`. Key calls include `replace`.
**CN:** 辅助函数 `_apply_draft_moe_backend` 封装了与 `apply draft moe 后端` 相关的可复用逻辑。 输入参数：`vllm_config`。 关键调用包括 `replace`。

### test_draft_model_moe_backend_override (lines 1053-1074)
```python
def test_draft_model_moe_backend_override():
    """When moe_backend is set in speculative_config, the draft VllmConfig
    should use it while the target keeps its own setting."""
    engine_args = EngineArgs(
        model="Qwen/Qwen3-1.7B",
        tensor_parallel_size=1,
        moe_backend="flashinfer_trtllm",
        speculative_config={
            "model": "Qwen/Qwen3-0.6B",
            "method": "draft_model",
            "num_speculative_tokens": 3,
            "moe_backend": "triton",
        },
    )
    tgt_config: VllmConfig = engine_args.create_engine_config()
    assert tgt_config.kernel_config.moe_backend == "flashinfer_trtllm"
    assert tgt_config.speculative_config.moe_backend == "triton"

    draft_config = _apply_draft_moe_backend(tgt_config)
    assert draft_config.kernel_config.moe_backend == "triton"
    # Target config must be unaffected.
    assert tgt_config.kernel_config.moe_backend == "flashinfer_trtllm"
```
**EN:** Test case covering `draft model moe backend override`. It exercises `EngineArgs, engine_args.create_engine_config, _apply_draft_moe_backend`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `draft model moe 后端 override` 的测试用例。 该测试会调用 `EngineArgs, engine_args.create_engine_config, _apply_draft_moe_backend`。 代码主体包含 4 个显式断言。

### test_draft_model_moe_backend_inherits_target (lines 1077-1096)
```python
def test_draft_model_moe_backend_inherits_target():
    """When moe_backend is not set in speculative_config, the draft should
    inherit the target's moe_backend."""
    engine_args = EngineArgs(
        model="Qwen/Qwen3-1.7B",
        tensor_parallel_size=1,
        moe_backend="flashinfer_cutlass",
        speculative_config={
            "model": "Qwen/Qwen3-0.6B",
            "method": "draft_model",
            "num_speculative_tokens": 3,
        },
    )
    tgt_config: VllmConfig = engine_args.create_engine_config()
    assert tgt_config.kernel_config.moe_backend == "flashinfer_cutlass"
    assert tgt_config.speculative_config.moe_backend is None

    draft_config = _apply_draft_moe_backend(tgt_config)
    assert draft_config.kernel_config.moe_backend == "flashinfer_cutlass"
    assert draft_config is tgt_config
```
**EN:** Test case covering `draft model moe backend inherits target`. It exercises `EngineArgs, engine_args.create_engine_config, _apply_draft_moe_backend`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `draft model moe 后端 inherits target` 的测试用例。 该测试会调用 `EngineArgs, engine_args.create_engine_config, _apply_draft_moe_backend`。 代码主体包含 4 个显式断言。

### test_draft_model_moe_backend_default_auto (lines 1099-1117)
```python
def test_draft_model_moe_backend_default_auto():
    """When neither target nor draft set moe_backend explicitly, both should
    default to 'auto'."""
    engine_args = EngineArgs(
        model="Qwen/Qwen3-1.7B",
        tensor_parallel_size=1,
        speculative_config={
            "model": "Qwen/Qwen3-0.6B",
            "method": "draft_model",
            "num_speculative_tokens": 3,
        },
    )
    tgt_config: VllmConfig = engine_args.create_engine_config()
    assert tgt_config.kernel_config.moe_backend == "auto"
    assert tgt_config.speculative_config.moe_backend is None

    draft_config = _apply_draft_moe_backend(tgt_config)
    assert draft_config.kernel_config.moe_backend == "auto"
    assert draft_config is tgt_config
```
**EN:** Test case covering `draft model moe backend default auto`. It exercises `EngineArgs, engine_args.create_engine_config, _apply_draft_moe_backend`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `draft model moe 后端 default auto` 的测试用例。 该测试会调用 `EngineArgs, engine_args.create_engine_config, _apply_draft_moe_backend`。 代码主体包含 4 个显式断言。

### test_draft_model_engine_args_rejects_invalid_tp_argname (lines 1120-1135)
```python
def test_draft_model_engine_args_rejects_invalid_tp_argname():
    """The user should pass "draft_tensor_parallel_size" rather than
    "tensor_parallel_size". We enforce this with validation."""

    engine_args = EngineArgs(
        model="Qwen/Qwen3-1.7B",
        tensor_parallel_size=1,
        speculative_config={
            "model": "Qwen/Qwen3-0.6B",
            "method": "draft_model",
            "num_speculative_tokens": 3,
            "tensor_parallel_size": 1,  # <<< invalid arg name
        },
    )
    with pytest.raises(ValueError):
        engine_args.create_engine_config()
```
**EN:** Test case covering `draft model engine args rejects invalid tp argname`. It exercises `EngineArgs, pytest.raises, engine_args.create_engine_config`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `draft model 引擎 args rejects invalid tp argname` 的测试用例。 该测试会调用 `EngineArgs, pytest.raises, engine_args.create_engine_config`。 主要通过预期异常检查来完成验证。

### assert_draft_model_correctness (lines 1138-1197)
```python
def assert_draft_model_correctness(args: ArgsTest):
    """Compare the outputs using and not using speculative decoding.
    In the greedy decoding case, the outputs must match EXACTLY."""
    test_prompts: list[Messages] = get_messages(
        dataset=args.dataset, n=args.num_prompts
    )

    spec_llm = LLM(
        model=args.target_model,
        speculative_config={
            "model": args.draft_model,
            "method": "draft_model",
            "num_speculative_tokens": args.num_speculative_tokens,
            "max_model_len": args.max_model_len,
            "enforce_eager": args.enforce_eager,
            "draft_tensor_parallel_size": args.draft_tensor_parallel_size,
            "parallel_drafting": args.parallel_drafting,
        },
    # ... excerpt omitted for brevity ...
        spec_llm, expected_accuracy_threshold=args.expected_gsm8k_accuracy
    assert acceptance_rate >= args.expected_acceptance_rate
    assert acceptance_len >= args.expected_acceptance_len
    cleanup_dist_env_and_memory()
    if not has_async:
        raise AsyncSchedulingNotEnabledError(
            "Expected async_scheduling=True for draft_model spec decode, got False."
            " See https://github.com/vllm-project/vllm/issues/38929"
        )
```
**EN:** Helper function `assert_draft_model_correctness` encapsulates reusable logic for `assert draft model correctness`. Inputs: `args`. Key calls include `get_messages, LLM, spec_llm.chat, spec_llm.get_metrics, compute_acceptance_rate, compute_acceptance_len`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `assert_draft_model_correctness` 封装了与 `assert draft model correctness` 相关的可复用逻辑。 输入参数：`args`。 关键调用包括 `get_messages, LLM, spec_llm.chat, spec_llm.get_metrics, compute_acceptance_rate, compute_acceptance_len`。 其中包含 2 个内部断言，用于保护前置假设。

### get_messages (lines 1200-1206)
```python
def get_messages(dataset: str, n: int) -> list[Messages]:
    if dataset == "test_prompts":
        return get_test_prompts(mm_enabled=False, num_prompts=n)
    elif dataset == "likaixin/InstructCoder":
        return get_instruct_coder_messages(n=n)
    else:
        raise NotImplementedError(f"Dataset '{dataset}' not implemented")
```
**EN:** Helper function `get_messages` encapsulates reusable logic for `messages`. Inputs: `dataset, n`. Key calls include `get_test_prompts, get_instruct_coder_messages, NotImplementedError`.
**CN:** 辅助函数 `get_messages` 封装了与 `messages` 相关的可复用逻辑。 输入参数：`dataset, n`。 关键调用包括 `get_test_prompts, get_instruct_coder_messages, NotImplementedError`。

### some_high_acceptance_metrics (lines 1209-1215)
```python
def some_high_acceptance_metrics() -> dict:
    return {
        "sampling_config": greedy_sampling(),
        "num_speculative_tokens": 3,
        "expected_acceptance_len": 3.4,  # ref: 3.75
        "expected_acceptance_rate": 0.8,  # ref: 0.9
    }
```
**EN:** Helper function `some_high_acceptance_metrics` encapsulates reusable logic for `some high acceptance metrics`. Key calls include `greedy_sampling`.
**CN:** 辅助函数 `some_high_acceptance_metrics` 封装了与 `some high acceptance metrics` 相关的可复用逻辑。 关键调用包括 `greedy_sampling`。

### compute_acceptance_rate (lines 1218-1234)
```python
def compute_acceptance_rate(
    metrics: list[Metric], prev_metrics: list[Metric] | None = None
) -> float:
    name2metric = {metric.name: metric for metric in metrics}
    n_draft_toks = name2metric["vllm:spec_decode_num_draft_tokens"].value
    if n_draft_toks == 0:
        return float("nan")
    n_accepted_toks = name2metric["vllm:spec_decode_num_accepted_tokens"].value
    if prev_metrics is not None:
        prev_name2metric = {metric.name: metric for metric in prev_metrics}
        n_draft_toks -= prev_name2metric["vllm:spec_decode_num_draft_tokens"].value
        n_accepted_toks -= prev_name2metric[
            "vllm:spec_decode_num_accepted_tokens"
        ].value
        if n_draft_toks <= 0:
            return float("nan")
    return n_accepted_toks / n_draft_toks
```
**EN:** Helper function `compute_acceptance_rate` encapsulates reusable logic for `compute acceptance rate`. Inputs: `metrics, prev_metrics`. Key calls include `float`.
**CN:** 辅助函数 `compute_acceptance_rate` 封装了与 `compute acceptance rate` 相关的可复用逻辑。 输入参数：`metrics, prev_metrics`。 关键调用包括 `float`。

### compute_acceptance_len (lines 1237-1253)
```python
def compute_acceptance_len(
    metrics: list[Metric], prev_metrics: list[Metric] | None = None
) -> float:
    name2metric = {metric.name: metric for metric in metrics}
    n_drafts = name2metric["vllm:spec_decode_num_drafts"].value
    n_accepted_toks = name2metric["vllm:spec_decode_num_accepted_tokens"].value
    if n_drafts == 0:
        return 1
    if prev_metrics is not None:
        prev_name2metric = {metric.name: metric for metric in prev_metrics}
        n_drafts -= prev_name2metric["vllm:spec_decode_num_drafts"].value
        n_accepted_toks -= prev_name2metric[
            "vllm:spec_decode_num_accepted_tokens"
        ].value
        if n_drafts <= 0:
            return 1
    return 1 + (n_accepted_toks / n_drafts)
```
**EN:** Helper function `compute_acceptance_len` encapsulates reusable logic for `compute acceptance len`. Inputs: `metrics, prev_metrics`.
**CN:** 辅助函数 `compute_acceptance_len` 封装了与 `compute acceptance len` 相关的可复用逻辑。 输入参数：`metrics, prev_metrics`。

### load_and_process_dataset (lines 1257-1278)
```python
def load_and_process_dataset(data_name: str):
    from datasets import load_dataset

    if data_name == "gsm8k":
        dataset = load_dataset("openai/gsm8k", "main", split="test")
        prompt_fmt = (
            "{question}\nPlease reason step by step,"
            " and put your final answer within \\boxed{{}}."
        )
        dataset = dataset.map(lambda x: {"turns": [prompt_fmt.format(**x)]})
    elif data_name == "mt-bench":
        dataset = load_dataset("HuggingFaceH4/mt_bench_prompts", split="train")
        dataset = dataset.map(lambda x: {"turns": x["prompt"]})
    elif data_name == "humaneval":
        dataset = load_dataset("openai/openai_humaneval", split="test")
        prompt_fmt = (
            "Write a solution to the following problem and make sure"
            " that it passes the tests:\n```python\n{prompt}\n```"
        )
        dataset = dataset.map(lambda x: {"turns": [prompt_fmt.format(**x)]})

    return dataset
```
**EN:** Helper function `load_and_process_dataset` encapsulates reusable logic for `load and process dataset`. Inputs: `data_name`. Key calls include `load_dataset, dataset.map, prompt_fmt.format`.
**CN:** 辅助函数 `load_and_process_dataset` 封装了与 `load and process dataset` 相关的可复用逻辑。 输入参数：`data_name`。 关键调用包括 `load_dataset, dataset.map, prompt_fmt.format`。

### dflash_config (lines 1282-1300)
```python
def dflash_config():
    target_model = "Qwen/Qwen3-8B"
    draft_model = "z-lab/Qwen3-8B-DFlash-b16"

    return dict(
        model=target_model,
        trust_remote_code=True,
        speculative_config={
            "method": "dflash",
            "model": draft_model,
            "num_speculative_tokens": 16,
            "max_model_len": 32768,
        },
        max_model_len=32768,
        max_num_seqs=128,
        gpu_memory_utilization=0.85,
        enforce_eager=False,
        disable_log_stats=False,
    )
```
**EN:** Fixture/helper `dflash_config` prepares reusable state for downstream tests. Key calls include `dict`.
**CN:** `dflash_config` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `dict`。

### test_dflash_acceptance_rates (lines 1303-1363)
```python
def test_dflash_acceptance_rates(dflash_config):
    """
    E2E test for DFlash (block diffusion) speculative decoding.
    Runs acceptance rate validation on GSM8k, MT-Bench, and HumanEval
    comparing against baseline results from the paper (Table 1).
    See https://github.com/z-lab/dflash/blob/main/benchmark_sglang.py for methodology.
    spec_llm = LLM(**dflash_config)

    max_prompts_per_dataset = 200  # mt-bench has 80, humaneval has 164, truncates gsm8k
    # All scores from Table 1 in https://arxiv.org/pdf/2602.06036
    expected_acceptance_lengths = {
        "mt-bench": 4.24,
        "humaneval": 6.50,
        "gsm8k": 6.54 * 0.95,  # runs with a subset of prompts so extra wide tol here
    }
    # ... excerpt omitted for brevity ...
    for dataset_name, expected_len in expected_acceptance_lengths.items():
        expected_len = expected_len * 0.9
            f" (expected at least {expected_len:.2f})"
        assert mean_acceptance_length >= expected_len, (
            f"DFlash acceptance_len for {dataset_name} is below expected threshold:"
            f"{mean_acceptance_length:.2f} < {expected_len:.2f}"
        )
    del spec_llm
    torch.accelerator.empty_cache()
    cleanup_dist_env_and_memory()
```
**EN:** Test case covering `dflash acceptance rates`. Inputs/fixtures: `dflash_config`. It exercises `LLM, spec_llm.get_tokenizer, expected_acceptance_lengths.items, accelerator.empty_cache, cleanup_dist_env_and_memory, load_and_process_dataset`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `dflash acceptance rates` 的测试用例。 输入或 fixture：`dflash_config`。 该测试会调用 `LLM, spec_llm.get_tokenizer, expected_acceptance_lengths.items, accelerator.empty_cache, cleanup_dist_env_and_memory, load_and_process_dataset`。 代码主体包含 1 个显式断言。

### test_synthetic_acceptance_rate (lines 1367-1411)
```python
def test_synthetic_acceptance_rate():
    """Verify that synthetic rejection sampling produces an acceptance
    length close to the requested mean acceptance length."""
    num_spec_tokens = 3
    expected_acceptance_len = 1.875
    tolerance = 0.15

    spec_llm = LLM(
        model="meta-llama/Llama-3.2-1B-Instruct",
        trust_remote_code=True,
        speculative_config={
            "method": "eagle3",
            "model": "nm-testing/Llama3_2_1B_speculator.eagle3",
            "num_speculative_tokens": num_spec_tokens,
            "max_model_len": 2048,
            "rejection_sample_method": "synthetic",
            "synthetic_acceptance_length": expected_acceptance_len,
        },
    # ... excerpt omitted for brevity ...
        f" (expected={expected_acceptance_len:.3f},"
    assert abs(acceptance_len - expected_acceptance_len) <= tolerance, (
        f" ±{tolerance} of expected {expected_acceptance_len:.3f}"
    )
    del spec_llm
    torch.accelerator.empty_cache()
    cleanup_dist_env_and_memory()
```
**EN:** Test case covering `synthetic acceptance rate`. It exercises `LLM, get_test_prompts, spec_llm.chat, spec_llm.get_metrics, compute_acceptance_len, print`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `synthetic acceptance rate` 的测试用例。 该测试会调用 `LLM, get_test_prompts, spec_llm.chat, spec_llm.get_metrics, compute_acceptance_len, print`。 代码主体包含 1 个显式断言。

### test_dflash_correctness (lines 1414-1440)
```python
def test_dflash_correctness(dflash_config):
    """
    E2E test for DFlash (block diffusion) speculative decoding.
    Ensures output correctness on GSM8k, with cudagraphs and batching on.
    """
    spec_llm = LLM(**dflash_config)

    # Evaluate GSM8k accuracy (Qwen3-8B ref: ~87-92% on GSM8k)
    evaluate_llm_for_gsm8k(spec_llm, expected_accuracy_threshold=0.8)

    current_metrics = spec_llm.get_metrics()
    acceptance_len = compute_acceptance_len(current_metrics)

    # AR is thoroughly validated in test_dflash_acceptance_rates, in a manner consistent
    # with the DFlash paper. However, that test measures AL per-request and thus runs
    # with a batch size of 1. To ensure that AL does not collapse with large batch sizes
    # we enforce a baseline on the AL over the full lm-eval-style GSM8k test.
    expected_len = 3.5  # Measured is 3.9 to 4.0
    print(f"DFlash GSM8k correctness test got AL {acceptance_len}")
    assert acceptance_len >= expected_len, (
        "DFlash correctness check failed with"
        f" {acceptance_len=}, expected at least {expected_len}"
    )

    del spec_llm
    torch.accelerator.empty_cache()
    cleanup_dist_env_and_memory()
```
**EN:** Test case covering `dflash correctness`. Inputs/fixtures: `dflash_config`. It exercises `LLM, evaluate_llm_for_gsm8k, spec_llm.get_metrics, compute_acceptance_len, print, accelerator.empty_cache`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `dflash correctness` 的测试用例。 输入或 fixture：`dflash_config`。 该测试会调用 `LLM, evaluate_llm_for_gsm8k, spec_llm.get_metrics, compute_acceptance_len, print, accelerator.empty_cache`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, tqdm, datasets, transformers, packaging.version`.
- **CN:** 外部库：`pytest, torch, tqdm, datasets, transformers, packaging.version`。
- **EN:** vLLM modules under test: `vllm, vllm.assets.base, vllm.assets.image, vllm.benchmarks.datasets, vllm.config, vllm.distributed, vllm.engine.arg_utils, vllm.platforms, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.assets.base, vllm.assets.image, vllm.benchmarks.datasets, vllm.config, vllm.distributed, vllm.engine.arg_utils, vllm.platforms, ...`。
- **EN:** Local test helpers: `tests.evals.gsm8k.gsm8k_eval, tests.utils`.
- **CN:** 本地测试辅助模块：`tests.evals.gsm8k.gsm8k_eval, tests.utils`。
- **EN:** Standard-library support: `os, random, collections.abc, dataclasses, typing`.
- **CN:** 标准库支持：`os, random, collections.abc, dataclasses, typing`。
