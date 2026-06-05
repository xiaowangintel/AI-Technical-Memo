# test_async_scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/e2e/general/test_async_scheduling.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises end-to-end `async scheduling` scenarios in the v1 test suite. / 在 v1 测试套件中覆盖端到端的 `async scheduling` 场景。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-21)
```python
import os
from itertools import repeat
from typing import Any

import pytest
import torch._dynamo.config as dynamo_config

from tests.utils import (
    large_gpu_mark,
    single_gpu_only,
)
from vllm import SamplingParams
from vllm.logprobs import Logprob
from vllm.platforms import current_platform
from vllm.sampling_params import StructuredOutputsParams
from vllm.v1.metrics.reader import Metric

from ....conftest import VllmRunner
from ....models.utils import check_outputs_equal
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch._dynamo.config`. vLLM modules under test include `vllm, vllm.logprobs, vllm.platforms, vllm.sampling_params, vllm.v1.metrics.reader`. Local helpers come from `tests.utils, tests.conftest, tests.models.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch._dynamo.config`。 被测试的 vLLM 模块包括 `vllm, vllm.logprobs, vllm.platforms, vllm.sampling_params, vllm.v1.metrics.reader`。 本地测试辅助逻辑来自 `tests.utils, tests.conftest, tests.models.utils`。

### Module state / 模块级状态 (lines 23-42)
```python
MODEL = "Qwen/Qwen3-0.6B"
MTP_MODEL = "meta-llama/Llama-3.2-1B-Instruct"

# Need to enforce eager for MRV2 while we sort out cudagraph issues.
ENFORCE_EAGER = os.getenv("ENFORCE_EAGER", "0") == "1"

first_prompt = (
    "The following numbers of the sequence "
    + ", ".join(str(i) for i in range(10))
    + " are:"
)
example_prompts = [first_prompt, "In one word, the capital of France is "] + [
    f"Tell me about the number {i}: " for i in range(32)
]

default_params = dict(
    temperature=0.0,  # greedy
    max_tokens=30,
    min_tokens=28,
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `MODEL, MTP_MODEL, ENFORCE_EAGER, first_prompt, example_prompts, default_params`. Shared setup calls include `dict, os.getenv, join, range, str`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`MODEL, MTP_MODEL, ENFORCE_EAGER, first_prompt, example_prompts, default_params`。 共享初始化调用包括 `dict, os.getenv, join, range, str`。

### test_without_spec_decoding (lines 46-104)
```python
def test_without_spec_decoding(
    sample_json_schema,
    monkeypatch: pytest.MonkeyPatch,
):
    """Test consistency of combos of async scheduling, preemption,
    uni/multiproc executor, prefill chunking."""
    struct_outputs = StructuredOutputsParams(json=sample_json_schema)
    test_sampling_params: list[dict[str, Any]] = [
        dict(),
        # dict(min_tokens=20),
        dict(frequency_penalty=-1.0),
        dict(bad_words=["the", " the"]),
        dict(logprobs=2),
        dict(logprobs=2, frequency_penalty=-1.0),
        dict(prompt_logprobs=2),
        dict(prompt_logprobs=2, logprobs=2),
        dict(structured_outputs=struct_outputs),
        dict(
    # ... excerpt omitted for brevity ...
        ]
        test_sampling_params = [
            p for p in test_sampling_params if p.get("structured_outputs") is not None

    run_tests(monkeypatch, MODEL, test_configs, test_sampling_params)
```
**EN:** Test case covering `without spec decoding`. Inputs/fixtures: `sample_json_schema, monkeypatch`. It exercises `StructuredOutputsParams, current_platform.is_rocm, run_tests, dict, p.get`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `without spec decoding` 的测试用例。 输入或 fixture：`sample_json_schema, monkeypatch`。 该测试会调用 `StructuredOutputsParams, current_platform.is_rocm, run_tests, dict, p.get`。 主要通过 mock、回调或输出检查来完成验证。

### test_with_eagle3_spec_decoding (lines 109-157)
```python
def test_with_eagle3_spec_decoding(sample_json_schema, monkeypatch: pytest.MonkeyPatch):
    """Test consistency and acceptance rates with some different combos of
    preemption, executor, async scheduling, prefill chunking,
    spec decoding model length.
    """

    spec_config = {
        "method": "eagle3",
        "num_speculative_tokens": 2,
        "model": "nm-testing/Llama3_2_1B_speculator.eagle3",
    }
    # Set small draft model len to force doesn't-fit-in-drafter case.
    spec_config_short = spec_config | {"max_model_len": 50}
    struct_outputs = StructuredOutputsParams(json=sample_json_schema)
    test_sampling_params = [
        dict(),
    # ... excerpt omitted for brevity ...
        (True, "uni", True, spec_config_short, False),
        (True, "mp", True, spec_config, True),
        (True, "uni", True, spec_config_short, True),
    ]
    run_tests(monkeypatch, MTP_MODEL, test_configs, test_sampling_params)
```
**EN:** Test case covering `with eagle3 spec decoding`. Inputs/fixtures: `sample_json_schema, monkeypatch`. It exercises `large_gpu_mark, StructuredOutputsParams, run_tests, dict`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `with eagle3 spec decoding` 的测试用例。 输入或 fixture：`sample_json_schema, monkeypatch`。 该测试会调用 `large_gpu_mark, StructuredOutputsParams, run_tests, dict`。 主要通过 mock、回调或输出检查来完成验证。

### test_with_ngram_gpu_spec_decoding (lines 161-194)
```python
def test_with_ngram_gpu_spec_decoding(monkeypatch: pytest.MonkeyPatch):
    """Test ngram_gpu speculative decoding with different configurations.

    This test specifically validates ngram_gpu behavior with various:
    - Number of speculative tokens (2-6)
    - Prompt lookup window sizes (min/max)
    - Async scheduling enabled (as in production)
    - Different executors and chunking settings
    """
    # Variant with larger speculation window
    ngram_gpu_config = {
        "method": "ngram_gpu",
        "num_speculative_tokens": 3,
        "prompt_lookup_max": 3,
        "prompt_lookup_min": 2,
    }
    # ... excerpt omitted for brevity ...
        (True, "mp", True, ngram_gpu_config, True),
    ]
    # Use MODEL (Qwen) for ngram_gpu tests as it's lighter weight
    # and ngram_gpu doesn't require a specific draft model
    run_tests(monkeypatch, MODEL, test_configs, [{}])
```
**EN:** Test case covering `with ngram GPU spec decoding`. Inputs/fixtures: `monkeypatch`. It exercises `mark.flaky, run_tests, current_platform.is_rocm`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `with ngram gpu spec decoding` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `mark.flaky, run_tests, current_platform.is_rocm`。 主要通过 mock、回调或输出检查来完成验证。

### run_tests (lines 198-315)
```python
def run_tests(
    monkeypatch: pytest.MonkeyPatch,
    model: str,
    test_configs: list[tuple],
    test_sampling_params: list[dict[str, Any]],
):
    """Test consistency of combos of async scheduling, preemption,
    uni/multiproc executor with spec decoding."""

    # Flex attention supports float32.
    attention_config = {"backend": "FLEX_ATTENTION"}
    with monkeypatch.context() as m:
        # lock matmul precision to full FP32 (IEEE)
        m.setenv("VLLM_FLOAT32_MATMUL_PRECISION", "highest")
        outputs: list[tuple[str, list, list]] = []
        for n, (
            test_preemption,
    # ... excerpt omitted for brevity ...
                    assert _all_logprobs_match(base_logprobs, test_logprobs)
                            assert (
                            # Currently the reported acceptance rate is expected to be
                            assert test_acceptance_rate > 0.1
                )
                if failure is None:
                    _, failure = reason
    if failure is not None:
        raise failure
```
**EN:** Helper function `run_tests` encapsulates reusable logic for `run tests`. Inputs: `monkeypatch, model, test_configs, test_sampling_params`. Key calls include `dynamo_config.patch, next, print, monkeypatch.context, m.setenv, enumerate`. It includes 3 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `run_tests` 封装了与 `run tests` 相关的可复用逻辑。 输入参数：`monkeypatch, model, test_configs, test_sampling_params`。 关键调用包括 `dynamo_config.patch, next, print, monkeypatch.context, m.setenv, enumerate`。 其中包含 3 个内部断言，用于保护前置假设。

### run_test (lines 318-405)
```python
def run_test(
    model: str,
    test_str: str,
    sampling_param_tests: list[dict[str, Any]],
    test_preemption: bool,
    executor: str,
    async_scheduling: bool,
    spec_config: dict[str, Any] | None,
    test_prefill_chunking: bool,
    attention_config: dict[str, Any] | None = None,
):
    spec_decoding = spec_config is not None
    cache_arg: dict[str, Any] = (
        # Force preemptions: with 32 blocks the cache holds at most a single
        # max-length request, so the ~34 concurrent prompts contend and trigger
        # preemption. (Prompts here are << max_model_len, so dropping
        # max_model_len from 4096 to 512 doesn't change generation behavior.)
        dict(num_gpu_blocks_override=32, max_model_len=512)
    # ... excerpt omitted for brevity ...
                assert preemptions > 0, "preemption test had no preemptions"
            with pytest.raises(AssertionError):
                    name_0=f"baseline params={params}",
                    name_1=f"other params={params}",
                )
                assert _all_logprobs_match(results[0][1], other_test_logprobs)

    return test_config, results, acceptance_rates
```
**EN:** Helper function `run_test` encapsulates reusable logic for `run test`. Inputs: `model, test_str, sampling_param_tests, test_preemption, executor, async_scheduling, spec_config, test_prefill_chunking, ...`. Key calls include `get, print, dict, VllmRunner, len, zip`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `run_test` 封装了与 `run test` 相关的可复用逻辑。 输入参数：`model, test_str, sampling_param_tests, test_preemption, executor, async_scheduling, spec_config, test_prefill_chunking, ...`。 关键调用包括 `get, print, dict, VllmRunner, len, zip`。 其中包含 2 个内部断言，用于保护前置假设。

### _all_logprobs_match (lines 408-417)
```python
def _all_logprobs_match(req_a, req_b) -> bool:
    return (
        req_a == req_b
        or len(req_a) == len(req_b)
        and all(
            len(seq_a) == len(seq_b)
            and all(_logprobs_match(a, b) for a, b in zip(seq_a, seq_b))
            for seq_a, seq_b in zip(req_a, req_b)
        )
    )
```
**EN:** Helper function `_all_logprobs_match` encapsulates reusable logic for `all logprobs match`. Inputs: `req_a, req_b`. Key calls include `all, len, zip, _logprobs_match`.
**CN:** 辅助函数 `_all_logprobs_match` 封装了与 `all 对数概率 match` 相关的可复用逻辑。 输入参数：`req_a, req_b`。 关键调用包括 `all, len, zip, _logprobs_match`。

### _logprobs_match (lines 420-436)
```python
def _logprobs_match(
    lps_a: dict[int, Logprob] | None,
    lps_b: dict[int, Logprob] | None,
) -> bool:
    if lps_a is None or lps_b is None:
        return lps_a is lps_b
    rel_tol, abs_tol = 1e-3, 1e-6
    return (
        len(lps_a) == len(lps_b)
        and lps_a.keys() == lps_b.keys()
        and all(
            a.decoded_token == b.decoded_token
            and a.rank == pytest.approx(b.rank, rel=0.005)
            and a.logprob == pytest.approx(b.logprob, rel=rel_tol, abs=abs_tol)
            for a, b in ((lps_a[x], lps_b[x]) for x in lps_a)
        )
    )
```
**EN:** Helper function `_logprobs_match` encapsulates reusable logic for `logprobs match`. Inputs: `lps_a, lps_b`. Key calls include `all, len, lps_a.keys, lps_b.keys, pytest.approx`.
**CN:** 辅助函数 `_logprobs_match` 封装了与 `对数概率 match` 相关的可复用逻辑。 输入参数：`lps_a, lps_b`。 关键调用包括 `all, len, lps_a.keys, lps_b.keys, pytest.approx`。

### _get_acceptance_rate (lines 439-442)
```python
def _get_acceptance_rate(before: list[Metric], after: list[Metric]) -> float:
    draft = _get_count(before, after, "vllm:spec_decode_num_draft_tokens")
    accept = _get_count(before, after, "vllm:spec_decode_num_accepted_tokens")
    return accept / draft if draft > 0 else 0.0
```
**EN:** Helper function `_get_acceptance_rate` encapsulates reusable logic for `acceptance rate`. Inputs: `before, after`. Key calls include `_get_count`.
**CN:** 辅助函数 `_get_acceptance_rate` 封装了与 `acceptance rate` 相关的可复用逻辑。 输入参数：`before, after`。 关键调用包括 `_get_count`。

### _get_count (lines 445-448)
```python
def _get_count(before: list[Metric], after: list[Metric], name: str) -> int:
    before_val = next(m.value for m in before if m.name == name)
    after_val = next(m.value for m in after if m.name == name)
    return after_val - before_val
```
**EN:** Helper function `_get_count` encapsulates reusable logic for `count`. Inputs: `before, after, name`. Key calls include `next`.
**CN:** 辅助函数 `_get_count` 封装了与 `count` 相关的可复用逻辑。 输入参数：`before, after, name`。 关键调用包括 `next`。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径
- **EN:** End-to-end integration rather than isolated units
- **CN:** 端到端集成而非孤立单元

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch._dynamo.config`.
- **CN:** 外部库：`pytest, torch._dynamo.config`。
- **EN:** vLLM modules under test: `vllm, vllm.logprobs, vllm.platforms, vllm.sampling_params, vllm.v1.metrics.reader`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.logprobs, vllm.platforms, vllm.sampling_params, vllm.v1.metrics.reader`。
- **EN:** Local test helpers: `tests.utils, tests.conftest, tests.models.utils`.
- **CN:** 本地测试辅助模块：`tests.utils, tests.conftest, tests.models.utils`。
- **EN:** Standard-library support: `os, itertools, typing`.
- **CN:** 标准库支持：`os, itertools, typing`。
