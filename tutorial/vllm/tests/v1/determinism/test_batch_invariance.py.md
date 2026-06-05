# test_batch_invariance.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/determinism/test_batch_invariance.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `batch invariance` behavior and regressions in the v1 stack. / 验证 v1 栈中 `批处理 invariance` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-18)
```python
import contextlib
import os
import random

import pytest
import torch
from utils import (
    BACKENDS,
    TEST_MODEL,
    _extract_step_logprobs,
    _random_prompt,
    skip_unsupported,
)

import vllm.envs as envs
from vllm import LLM, SamplingParams
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, utils`. vLLM modules under test include `vllm.envs, vllm`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, utils`。 被测试的 vLLM 模块包括 `vllm.envs, vllm`。

### test_v1_generation_is_deterministic_across_batch_sizes_with_needle (lines 27-142)
```python
def test_v1_generation_is_deterministic_across_batch_sizes_with_needle(
    backend,
):
    """
    Ensures that the same request (the 'needle' prompt) yields identical output
    whether run alone (bs=1) or mixed into a larger batch (e.g., bs=64),
    using the high-level v1 LLM() API only (no manual batching).

    Strategy:
    - Create a single LLM engine configured for the larger batch limit (N).
    - Compute a baseline output for the needle prompt when it is run alone.
    - For many trials, generate a mixed batch (size N) where the needle appears
      at a random position among random filler prompts using the same engine.
    - Track how many trials match vs mismatch, and report totals at the end.
      The test fails if any mismatches occur, but we still dump pass/fail
      counts.
    Notes:
    # ... excerpt omitted for brevity ...
    assert max_batch_size >= 2, "Batch size should be >= 2 to mix needle."
        assert len(baseline_out) == 1
        assert len(baseline_out[0].outputs) >= 1
            assert needle_output.prompt == needle_prompt
            assert len(needle_output.outputs) >= 1
    finally:
        # Ensure engines are shutdown to free GPU/VRAM across test sessions
        if llm is not None:
            with contextlib.suppress(Exception):
                llm.shutdown()
```
**EN:** Parameterized test covering `v1 generation is deterministic across batch sizes with needle`. Parameter axes: `backend`. Inputs/fixtures: `backend`. It exercises `mark.timeout, mark.parametrize, int, random.seed, float, SamplingParams`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `v1 generation is deterministic across 批处理 sizes with needle` 的测试用例。 参数维度：`backend`。 输入或 fixture：`backend`。 该测试会调用 `mark.timeout, mark.parametrize, int, random.seed, float, SamplingParams`。 代码主体包含 5 个显式断言。

### test_logprobs_bitwise_batch_invariance_bs1_vs_bsN (lines 154-374)
```python
def test_logprobs_bitwise_batch_invariance_bs1_vs_bsN(
    backend,
    block_m,
    block_n,
):
    seed = int(os.getenv("VLLM_TEST_SEED", "12345"))
    random.seed(seed)
    tp_size = int(os.getenv("VLLM_TEST_TP_SIZE", "1"))

    # For batch invariance, disable custom all-reduce to ensure deterministic
    # all-reduce operations (custom all-reduce may not be deterministic)
    import vllm.envs as envs
    disable_custom_ar = envs.VLLM_BATCH_INVARIANT
    if disable_custom_ar:
        print(f"\n{'=' * 80}")
        print(f"BATCH INVARIANCE MODE: Disabling custom all-reduce (TP={tp_size})")
    # ... excerpt omitted for brevity ...
        assert len(outs) == 1
    assert len(outs_batched) == len(prompts)
        # Fail the test with summary
        msg = (
            f"Batch invariance violated in {len(failed_prompts)}/"
            f"{len(prompts)} prompts. See output above for details."
        )
        pytest.fail(msg)
```
**EN:** Parameterized test covering `logprobs bitwise batch invariance bs1 vs bsN`. Parameter axes: `backend, block_m, block_n`. Inputs/fixtures: `backend, block_m, block_n`. It exercises `mark.parametrize, int, random.seed, LLM, SamplingParams, print`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `对数概率 bitwise 批处理 invariance bs1 vs bsn` 的测试用例。 参数维度：`backend, block_m, block_n`。 输入或 fixture：`backend, block_m, block_n`。 该测试会调用 `mark.parametrize, int, random.seed, LLM, SamplingParams, print`。 代码主体包含 2 个显式断言。

### test_simple_generation (lines 382-424)
```python
def test_simple_generation(backend):
    """
    Simple test that runs the model with a basic prompt and prints the output.
    Useful for quick smoke testing and debugging.
    model = TEST_MODEL

    llm = LLM(
        model=model,
        max_num_seqs=1,
        tensor_parallel_size=int(os.getenv("VLLM_TP_SIZE", "1")),
        gpu_memory_utilization=0.9,
        max_model_len=2048,
        dtype="auto",
        enable_prefix_caching=False,
        attention_config={"backend": backend},
    )
    # ... excerpt omitted for brevity ...
        assert len(outputs) == 1
        print(f"Full completion: '{prompt}{output_text}'")
        print(f"{'=' * 80}\n")
    finally:
        with contextlib.suppress(Exception):
            llm.shutdown()
```
**EN:** Parameterized test covering `simple generation`. Parameter axes: `backend`. Inputs/fixtures: `backend`. It exercises `mark.parametrize, LLM, SamplingParams, print, llm.generate, int`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `simple generation` 的测试用例。 参数维度：`backend`。 输入或 fixture：`backend`。 该测试会调用 `mark.parametrize, LLM, SamplingParams, print, llm.generate, int`。 代码主体包含 1 个显式断言。

### test_logprobs_without_batch_invariance_should_fail (lines 432-640)
```python
def test_logprobs_without_batch_invariance_should_fail(
    backend, monkeypatch: pytest.MonkeyPatch
):
    """
    This test is the inverse of test_logprobs_bitwise_batch_invariance_bs1_vs_bsN.
    It DISABLES batch invariance mode and expects to see non-deterministic behavior
    between BS=1 and BS=N runs. This demonstrates that batch invariance is actually
    doing something useful.

    The test will PASS if we detect differences (proving batch invariance matters).
    The test will FAIL if everything matches (suggesting batch invariance isn't needed).
    # CRITICAL: Disable batch invariance for this test
    monkeypatch.setenv("VLLM_BATCH_INVARIANT", "0")
    monkeypatch.setattr(envs, "VLLM_BATCH_INVARIANT", False)
    seed = int(os.getenv("VLLM_TEST_SEED", "12345"))
    random.seed(seed)
    tp_size = int(os.getenv("VLLM_TEST_TP_SIZE", "1"))
    # ... excerpt omitted for brevity ...
        assert len(outs) == 1
    assert len(outs_batched) == len(prompts)
            f"This suggests batch invariance might not be necessary, "
            f"or the test needs more sensitive prompts."
        )
        print(fail_msg)
        print(f"{'=' * 80}\n")
        pytest.fail(fail_msg)
```
**EN:** Parameterized test covering `logprobs without batch invariance should fail`. Parameter axes: `backend`. Inputs/fixtures: `backend, monkeypatch`. It exercises `mark.parametrize, monkeypatch.setenv, monkeypatch.setattr, int, random.seed, print`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `对数概率 without 批处理 invariance should fail` 的测试用例。 参数维度：`backend`。 输入或 fixture：`backend, monkeypatch`。 该测试会调用 `mark.parametrize, monkeypatch.setenv, monkeypatch.setattr, int, random.seed, print`。 代码主体包含 2 个显式断言。

### test_decode_logprobs_match_prefill_logprobs (lines 645-904)
```python
def test_decode_logprobs_match_prefill_logprobs(
    backend,
):
    """
    Test that verifies decode logprobs match prefill logprobs.

    For each decoded token at position i:
    1. Run decode to generate N tokens and collect their logprobs
    2. For each position i in [0, N):
       - Take prefix = prompt + tokens[0:i]
       - Run prefill(prefix + tokens[i]) to get logprob of tokens[i]
       - Verify prefill logprob matches decode logprob bitwise
    This ensures that the logprobs from decode are consistent with what
    we would get if we ran prefill on each prefix.
    seed = int(os.getenv("VLLM_TEST_SEED", "12345"))
    random.seed(seed)
    # ... excerpt omitted for brevity ...
            f"Decode logprobs do not match prefill logprobs: "
            f"{len(failed_comparisons)} mismatches found."
        )
    else:
        print("✓ SUCCESS: All decode logprobs match prefill logprobs bitwise!")
        print(f"{'=' * 80}\n")
```
**EN:** Parameterized test covering `decode logprobs match prefill logprobs`. Parameter axes: `backend`. Inputs/fixtures: `backend`. It exercises `mark.parametrize, int, random.seed, LLM, SamplingParams, print`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `decode 对数概率 match prefill 对数概率` 的测试用例。 参数维度：`backend`。 输入或 fixture：`backend`。 该测试会调用 `mark.parametrize, int, random.seed, LLM, SamplingParams, print`。 主要通过 mock、回调或输出检查来完成验证。

### LLM_with_max_seqs (lines 907-929)
```python
def LLM_with_max_seqs(
    model: str,
    max_num_seqs: int,
    gpu_memory_utilization: float,
    max_model_len: int,
    attention_config: dict | None = None,
) -> LLM:
    """
    Helper to construct an LLM with a specific max_num_seqs (batch-size limit)
    using the high-level v1 LLM API, while constraining memory usage.
    """
    return LLM(
        model=model,
        max_num_seqs=max_num_seqs,
        gpu_memory_utilization=gpu_memory_utilization,
        max_model_len=max_model_len,
        dtype="auto",
        tensor_parallel_size=int(os.getenv("VLLM_TP_SIZE", "1")),
        enable_prefix_caching=False,
        attention_config=attention_config,
        # Enable for MOE models
        # enable_expert_parallel=True,
    )
```
**EN:** Helper function `LLM_with_max_seqs` encapsulates reusable logic for `LLM with max seqs`. Inputs: `model, max_num_seqs, gpu_memory_utilization, max_model_len, attention_config`. Key calls include `LLM, int, os.getenv`.
**CN:** 辅助函数 `LLM_with_max_seqs` 封装了与 `llm with max seqs` 相关的可复用逻辑。 输入参数：`model, max_num_seqs, gpu_memory_utilization, max_model_len, attention_config`。 关键调用包括 `LLM, int, os.getenv`。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, utils`.
- **CN:** 外部库：`pytest, torch, utils`。
- **EN:** vLLM modules under test: `vllm.envs, vllm`.
- **CN:** 被测试的 vLLM 模块：`vllm.envs, vllm`。
- **EN:** Standard-library support: `contextlib, os, random, struct`.
- **CN:** 标准库支持：`contextlib, os, random, struct`。
