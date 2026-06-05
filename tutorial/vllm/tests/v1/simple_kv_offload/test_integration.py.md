# test_integration.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/simple_kv_offload/test_integration.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Integration tests for SimpleCPUOffloadConnector with real models. / 该文件的文档字符串表明其用途：`integration tests for simplecpuoffloadconnector with real models`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Integration tests for SimpleCPUOffloadConnector with real models."""
```
**EN:** Module docstring that declares the scope of the file: Integration tests for SimpleCPUOffloadConnector with real models.
**CN:** 模块文档字符串直接说明了文件范围：`integration tests for simplecpuoffloadconnector with real models`。

### Imports and setup / 导入与设置 (lines 5-11)
```python
import time

import pytest

from vllm import LLM, SamplingParams, TokensPrompt
from vllm.config import KVTransferConfig
from vllm.platforms import current_platform
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm, vllm.config, vllm.platforms`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.platforms`。

### Module state / 模块级状态 (lines 13-26)
```python
if not current_platform.is_cuda_alike():
    pytest.skip("Requires CUDA or ROCm", allow_module_level=True)

# Small models for default CI / local runs (accuracy only).
SMALL_MODELS = [
    "meta-llama/Llama-3.2-1B-Instruct",
    "google/gemma-3-1b-it",
]

# Large models for optional perf runs only (slow to load and execute).
PERF_MODELS = [
    "meta-llama/Llama-3.1-8B",
    "openai/gpt-oss-20b",
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `SMALL_MODELS, PERF_MODELS`. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `current_platform.is_cuda_alike, pytest.skip`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`SMALL_MODELS, PERF_MODELS`。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `current_platform.is_cuda_alike, pytest.skip`。

### _make_llm (lines 29-44)
```python
def _make_llm(model: str, lazy: bool, cpu_bytes_to_use: int) -> LLM:
    kv_transfer_config = KVTransferConfig(
        kv_connector="SimpleCPUOffloadConnector",
        kv_role="kv_both",
        kv_connector_extra_config={
            "cpu_bytes_to_use": cpu_bytes_to_use,
            "lazy_offload": lazy,
        },
    )
    return LLM(
        model=model,
        kv_cache_memory_bytes=40 << 30,  # 40 GiB
        disable_hybrid_kv_cache_manager=False,
        enable_prefix_caching=True,
        kv_transfer_config=kv_transfer_config,
    )
```
**EN:** Helper function `_make_llm` encapsulates reusable logic for `LLM`. Inputs: `model, lazy, cpu_bytes_to_use`. Key calls include `KVTransferConfig, LLM`.
**CN:** 辅助函数 `_make_llm` 封装了与 `llm` 相关的可复用逻辑。 输入参数：`model, lazy, cpu_bytes_to_use`。 关键调用包括 `KVTransferConfig, LLM`。

### _flush_gpu_cache (lines 47-71)
```python
def _flush_gpu_cache(llm: LLM, sampling_params: SamplingParams, seed: int = 0):
    """Generate enough filler requests to allocate the entire GPU KV cache.

    This pushes all prior blocks through the free queue so that the lazy
    cursor offloads them to CPU before they are evicted.
    """
    cache_config = llm.llm_engine.vllm_config.cache_config
    num_gpu_blocks = cache_config.num_gpu_blocks
    block_size = cache_config.block_size
    # Use 1.2x GPU capacity to give the lazy cursor enough scheduling steps
    # to walk past all target blocks near the tail of the free queue.
    total_tokens_needed = int(num_gpu_blocks * block_size * 1.5)

    # Use token-id prompts so each filler is unique (no prefix sharing).
    # Split into multiple requests to stay under max_model_len.
    max_tokens_per_req = 4096
    num_fillers = (total_tokens_needed + max_tokens_per_req - 1) // max_tokens_per_req
    batch_size = 10
    for i in range(0, num_fillers, batch_size):
        batch_end = min(i + batch_size, num_fillers)
        filler_prompts = []
        for j in range(i, batch_end):
            ids = [seed * num_fillers + j + 1] * max_tokens_per_req
            filler_prompts.append(TokensPrompt(prompt_token_ids=ids))
        llm.generate(filler_prompts, sampling_params, use_tqdm=False)
```
**EN:** Helper function `_flush_gpu_cache` encapsulates reusable logic for `flush GPU cache`. Inputs: `llm, sampling_params, seed`. Key calls include `int, range, min, llm.generate, filler_prompts.append, TokensPrompt`.
**CN:** 辅助函数 `_flush_gpu_cache` 封装了与 `flush gpu 缓存` 相关的可复用逻辑。 输入参数：`llm, sampling_params, seed`。 关键调用包括 `int, range, min, llm.generate, filler_prompts.append, TokensPrompt`。

### _accuracy_test (lines 74-101)
```python
def _accuracy_test(llm: LLM, lazy: bool = False):
    """Verify that CPU-loaded KV produces correct output."""
    sampling_params = SamplingParams(max_tokens=1, temperature=0)
    prompt = "hi " * 2000 + "Let's count to ten. One, two, three, "

    # Cold run — populate GPU cache and trigger CPU offload
    cold_output = llm.generate(prompt, sampling_params, use_tqdm=False)[0]

    # CPU hit runs
    test_count = 10
    success_count = 0
    expected = cold_output.outputs[0].text
    for i in range(test_count):
        if lazy:
            _flush_gpu_cache(llm, sampling_params, seed=i)
        time.sleep(2)  # let engine core drain pending transfers

        # Reset GPU prefix cache so next run must load from CPU
        if not llm.reset_prefix_cache():
            print(f"GPU prefix cache reset failed for iteration {i}")

        output = llm.generate(prompt, sampling_params, use_tqdm=False)[0]
        if output.outputs[0].text == expected:
            success_count += 1

    assert success_count >= 0.5 * test_count, (
        f"Accuracy too low: {success_count}/{test_count} matched '{expected}'"
    )
```
**EN:** Helper function `_accuracy_test` encapsulates reusable logic for `accuracy test`. Inputs: `llm, lazy`. Key calls include `SamplingParams, range, llm.generate, time.sleep, _flush_gpu_cache, llm.reset_prefix_cache`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_accuracy_test` 封装了与 `accuracy test` 相关的可复用逻辑。 输入参数：`llm, lazy`。 关键调用包括 `SamplingParams, range, llm.generate, time.sleep, _flush_gpu_cache, llm.reset_prefix_cache`。 其中包含 1 个内部断言，用于保护前置假设。

### _latency_test (lines 104-143)
```python
def _latency_test(llm: LLM, lazy: bool = False):
    """Verify CPU cache hit is faster than cold compute."""
    sampling_params = SamplingParams(max_tokens=1, seed=42)
    prompt_token_ids = [0] * 10001

    num_times_cpu_better = 0
    num_tests = 10
    for i in range(num_tests):
        prompt_token_ids[0] = i
        prompts = [TokensPrompt(prompt_token_ids=prompt_token_ids)]
        # Cold
        time.sleep(2)  # let engine core drain pending transfers
        if not llm.reset_prefix_cache():
            print(f"GPU prefix cache reset failed for iteration {i}")
        start = time.time()
        llm.generate(prompts, sampling_params, use_tqdm=False)
        cold_time = time.time() - start
    # ... excerpt omitted for brevity ...
        if cpu_time < cold_time:
            num_times_cpu_better += 1
    assert num_times_cpu_better >= 0.8 * num_tests, (
        f"CPU hit only faster {num_times_cpu_better}/{num_tests} times"
    )
```
**EN:** Helper function `_latency_test` encapsulates reusable logic for `latency test`. Inputs: `llm, lazy`. Key calls include `SamplingParams, range, time.sleep, time.time, llm.generate, TokensPrompt`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_latency_test` 封装了与 `latency test` 相关的可复用逻辑。 输入参数：`llm, lazy`。 关键调用包括 `SamplingParams, range, time.sleep, time.time, llm.generate, TokensPrompt`。 其中包含 1 个内部断言，用于保护前置假设。

### test_simple_cpu_offload_accuracy (lines 149-155)
```python
def test_simple_cpu_offload_accuracy(model: str):
    """Store to CPU, reset GPU, load from CPU; verify output matches baseline."""
    llm = _make_llm(model, False, 1 << 30)  # 1GB
    try:
        _accuracy_test(llm, lazy=False)
    finally:
        del llm
```
**EN:** Parameterized test covering `simple CPU offload accuracy`. Parameter axes: `model`. Inputs/fixtures: `model`. It exercises `mark.parametrize, _make_llm, _accuracy_test`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `simple cpu offload accuracy` 的测试用例。 参数维度：`model`。 输入或 fixture：`model`。 该测试会调用 `mark.parametrize, _make_llm, _accuracy_test`。 主要通过 mock、回调或输出检查来完成验证。

### test_simple_cpu_offload_perf_latency (lines 161-167)
```python
def test_simple_cpu_offload_perf_latency(model: str):
    """CPU KV hit should beat cold prefill on long context (large models only)."""
    llm = _make_llm(model, False, 10 << 30)  # 10GB
    try:
        _latency_test(llm, lazy=False)
    finally:
        del llm
```
**EN:** Parameterized test covering `simple CPU offload perf latency`. Parameter axes: `model`. Inputs/fixtures: `model`. It exercises `mark.parametrize, _make_llm, _latency_test`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `simple cpu offload perf latency` 的测试用例。 参数维度：`model`。 输入或 fixture：`model`。 该测试会调用 `mark.parametrize, _make_llm, _latency_test`。 主要通过 mock、回调或输出检查来完成验证。

### test_simple_cpu_offload_accuracy_lazy (lines 173-180)
```python
def test_simple_cpu_offload_accuracy_lazy(model: str):
    """Lazy mode: flush GPU cache to trigger CPU offload, then verify hit."""
    # CPU must be larger than GPU KV cache to avoid evicting offloaded blocks.
    llm = _make_llm(model, True, 80 << 30)  # 80GB
    try:
        _accuracy_test(llm, lazy=True)
    finally:
        del llm
```
**EN:** Parameterized test covering `simple CPU offload accuracy lazy`. Parameter axes: `model`. Inputs/fixtures: `model`. It exercises `mark.parametrize, _make_llm, _accuracy_test`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `simple cpu offload accuracy lazy` 的测试用例。 参数维度：`model`。 输入或 fixture：`model`。 该测试会调用 `mark.parametrize, _make_llm, _accuracy_test`。 主要通过 mock、回调或输出检查来完成验证。

### test_simple_cpu_offload_perf_latency_lazy (lines 186-193)
```python
def test_simple_cpu_offload_perf_latency_lazy(model: str):
    """Lazy mode: CPU KV hit should beat cold prefill (large models only)."""
    # CPU must be larger than GPU KV cache to avoid evicting offloaded blocks.
    llm = _make_llm(model, True, 80 << 30)  # 80GB
    try:
        _latency_test(llm, lazy=True)
    finally:
        del llm
```
**EN:** Parameterized test covering `simple CPU offload perf latency lazy`. Parameter axes: `model`. Inputs/fixtures: `model`. It exercises `mark.parametrize, _make_llm, _latency_test`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `simple cpu offload perf latency lazy` 的测试用例。 参数维度：`model`。 输入或 fixture：`model`。 该测试会调用 `mark.parametrize, _make_llm, _latency_test`。 主要通过 mock、回调或输出检查来完成验证。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.platforms`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.platforms`。
- **EN:** Standard-library support: `time`.
- **CN:** 标准库支持：`time`。
