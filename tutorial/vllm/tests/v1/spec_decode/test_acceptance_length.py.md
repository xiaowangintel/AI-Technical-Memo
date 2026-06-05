# test_acceptance_length.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_acceptance_length.py`
- **Repository**: vllm-project/vllm
- **Purpose**: EAGLE3 Acceptance Length Regression Tests. / 该文件的文档字符串表明其用途：`eagle3 acceptance length regression tests`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-10)
```python
"""
EAGLE3 Acceptance Length Regression Tests.

These tests verify that acceptance lengths for EAGLE3 speculative decoding
do not regress across vLLM commits. Each test runs inference on the MT-Bench
dataset and asserts that the mean acceptance length is within tolerance of
the expected baseline.
"""
```
**EN:** Module docstring that declares the scope of the file: EAGLE3 Acceptance Length Regression Tests.
**CN:** 模块文档字符串直接说明了文件范围：`eagle3 acceptance length regression tests`。

### Imports and setup / 导入与设置 (lines 12-26)
```python
from dataclasses import dataclass, field
from types import SimpleNamespace

import pytest
import torch

from tests.conftest import VllmRunner
from tests.utils import large_gpu_mark
from vllm import SamplingParams
from vllm.benchmarks.datasets import get_samples
from vllm.inputs import TokensPrompt
from vllm.platforms import current_platform
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.attention.selector import AttentionSelectorConfig
from vllm.v1.metrics.reader import Counter, Vector
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm, vllm.benchmarks.datasets, vllm.inputs, vllm.platforms, vllm.v1.attention.backends.registry, ...`. Local helpers come from `tests.conftest, tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm, vllm.benchmarks.datasets, vllm.inputs, vllm.platforms, vllm.v1.attention.backends.registry, ...`。 本地测试辅助逻辑来自 `tests.conftest, tests.utils`。

### Eagle3ModelConfig (lines 30-41)
```python
class Eagle3ModelConfig:
    verifier: str
    drafter: str
    expected_acceptance_length: float
    expected_acceptance_lengths_per_pos: list[float] = field(default_factory=list)
    id: str = ""
    # Backends that are incompatible with this model (will be skipped)
    excluded_backends: set[AttentionBackendEnum] = field(default_factory=set)
    # Pytest marks for this configuration
    marks: list = field(default_factory=list)
    # Custom relative tolerance (defaults to DEFAULT_RTOL if None)
    rtol: float | None = None
```
**EN:** Class `Eagle3ModelConfig` groups 0 test method(s).
**CN:** 类 `Eagle3ModelConfig` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 48-98)
```python
EAGLE3_MODEL_CONFIGS = [
    Eagle3ModelConfig(
        verifier="meta-llama/Llama-3.1-8B-Instruct",
        drafter="RedHatAI/Llama-3.1-8B-Instruct-speculator.eagle3",
        expected_acceptance_length=2.60,
        expected_acceptance_lengths_per_pos=[0.7296, 0.5208, 0.3545],
        id="llama3-8b-eagle3",
    ),
        verifier="Qwen/Qwen3-8B",
        drafter="RedHatAI/Qwen3-8B-speculator.eagle3",
        expected_acceptance_length=2.26,
        expected_acceptance_lengths_per_pos=[0.6541, 0.3993, 0.2020],
        id="qwen3-8b-eagle3",
        verifier="openai/gpt-oss-20b",
        drafter="RedHatAI/gpt-oss-20b-speculator.eagle3",
    # ... excerpt omitted for brevity ...
        expected_acceptance_length=2.56,
        expected_acceptance_lengths_per_pos=[0.7165, 0.5120, 0.3337],
        expected_acceptance_length=1.35,
        expected_acceptance_lengths_per_pos=[0.2900, 0.0620, 0.0115],
# TP sizes to test
TP_SIZES = [1, 2, 4]

# Backends excluded from testing due to significantly different behavior
EXCLUDED_BACKENDS = {AttentionBackendEnum.FLEX_ATTENTION}
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `EAGLE3_MODEL_CONFIGS, DEFAULT_NUM_SPEC_TOKENS, DEFAULT_NUM_PROMPTS, DEFAULT_OUTPUT_LEN, DEFAULT_MAX_MODEL_LEN, DEFAULT_RTOL, ...`. Shared setup calls include `Eagle3ModelConfig`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`EAGLE3_MODEL_CONFIGS, DEFAULT_NUM_SPEC_TOKENS, DEFAULT_NUM_PROMPTS, DEFAULT_OUTPUT_LEN, DEFAULT_MAX_MODEL_LEN, DEFAULT_RTOL, ...`。 共享初始化调用包括 `Eagle3ModelConfig`。

### get_available_attention_backends (lines 101-137)
```python
def get_available_attention_backends() -> list[str]:
    # Check if get_valid_backends is actually defined in the platform class
    # (not just returning None from __getattr__)
    get_valid_backends = getattr(current_platform.__class__, "get_valid_backends", None)
    if get_valid_backends is None:
        if current_platform.is_rocm():
            # ROCm uses Triton as its default attention backend since
            # Flash Attention is not supported.
            return ["TRITON_ATTN"]
        else:
            return ["FLASH_ATTN"]

    device_capability = current_platform.get_device_capability()
    if device_capability is None:
        return ["FLASH_ATTN"]
    attn_selector_config = AttentionSelectorConfig(
        head_size=128,
    # ... excerpt omitted for brevity ...
    return [
        backend.name
        for backend, _ in valid_backends
        if backend not in EXCLUDED_BACKENDS
    ]
```
**EN:** Helper function `get_available_attention_backends` encapsulates reusable logic for `available attention backends`. Key calls include `getattr, current_platform.get_device_capability, AttentionSelectorConfig, current_platform.get_valid_backends, current_platform.is_rocm`.
**CN:** 辅助函数 `get_available_attention_backends` 封装了与 `available 注意力 backends` 相关的可复用逻辑。 关键调用包括 `getattr, current_platform.get_device_capability, AttentionSelectorConfig, current_platform.get_valid_backends, current_platform.is_rocm`。

### get_attention_backend_params (lines 140-141)
```python
def get_attention_backend_params() -> list[str]:
    return get_available_attention_backends()
```
**EN:** Helper function `get_attention_backend_params` encapsulates reusable logic for `attention backend params`. Key calls include `get_available_attention_backends`.
**CN:** 辅助函数 `get_attention_backend_params` 封装了与 `注意力 后端 params` 相关的可复用逻辑。 关键调用包括 `get_available_attention_backends`。

### get_tp_size_params (lines 144-146)
```python
def get_tp_size_params() -> list[pytest.param]:
    num_gpus = torch.accelerator.device_count() if torch.cuda.is_available() else 1
    return [pytest.param(tp, id=f"tp{tp}") for tp in TP_SIZES if tp <= num_gpus]
```
**EN:** Helper function `get_tp_size_params` encapsulates reusable logic for `tp size params`. Key calls include `cuda.is_available, accelerator.device_count, pytest.param`.
**CN:** 辅助函数 `get_tp_size_params` 封装了与 `tp size params` 相关的可复用逻辑。 关键调用包括 `cuda.is_available, accelerator.device_count, pytest.param`。

### get_mt_bench_prompts (lines 149-175)
```python
def get_mt_bench_prompts(
    tokenizer, num_prompts: int = DEFAULT_NUM_PROMPTS
) -> list[list[int]]:
    args = SimpleNamespace(
        dataset_name="hf",
        dataset_path="philschmid/mt-bench",
        num_prompts=num_prompts,
        seed=42,
        no_oversample=False,
        endpoint_type="openai-chat",
        input_len=None,
        output_len=DEFAULT_OUTPUT_LEN,
        sharegpt_output_len=DEFAULT_OUTPUT_LEN,
        hf_name=None,
        hf_split="train",
        hf_subset=None,
        hf_output_len=DEFAULT_OUTPUT_LEN,
        no_stream=True,
        disable_shuffle=False,
        skip_chat_template=False,
        trust_remote_code=False,
    )
    samples = get_samples(args, tokenizer)
    prompt_ids = [
        tokenizer.encode(sample.prompt, add_special_tokens=False) for sample in samples
    ]
    return prompt_ids
```
**EN:** Helper function `get_mt_bench_prompts` encapsulates reusable logic for `mt bench prompts`. Inputs: `tokenizer, num_prompts`. Key calls include `SimpleNamespace, get_samples, tokenizer.encode`.
**CN:** 辅助函数 `get_mt_bench_prompts` 封装了与 `mt bench prompts` 相关的可复用逻辑。 输入参数：`tokenizer, num_prompts`。 关键调用包括 `SimpleNamespace, get_samples, tokenizer.encode`。

### extract_acceptance_metrics (lines 178-210)
```python
def extract_acceptance_metrics(metrics, num_spec_tokens: int) -> dict:
    num_drafts = 0
    num_accepted_tokens = 0
    acceptance_counts = [0] * num_spec_tokens

    for metric in metrics:
        if metric.name == "vllm:spec_decode_num_drafts":
            assert isinstance(metric, Counter)
            num_drafts += metric.value
        elif metric.name == "vllm:spec_decode_num_accepted_tokens":
            num_accepted_tokens += metric.value
        elif metric.name == "vllm:spec_decode_num_accepted_tokens_per_pos":
            assert isinstance(metric, Vector)
            for pos in range(min(len(metric.values), num_spec_tokens)):
                acceptance_counts[pos] += metric.values[pos]
    # Calculate mean acceptance length
    # ... excerpt omitted for brevity ...
    return {
        "acceptance_length": acceptance_length,
        "acceptance_lengths_per_pos": acceptance_lengths_per_pos,
        "num_drafts": num_drafts,
        "num_accepted_tokens": num_accepted_tokens,
    }
```
**EN:** Helper function `extract_acceptance_metrics` encapsulates reusable logic for `extract acceptance metrics`. Inputs: `metrics, num_spec_tokens`. Key calls include `isinstance, range, min, len`. It includes 3 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `extract_acceptance_metrics` 封装了与 `extract acceptance metrics` 相关的可复用逻辑。 输入参数：`metrics, num_spec_tokens`。 关键调用包括 `isinstance, range, min, len`。 其中包含 3 个内部断言，用于保护前置假设。

### test_eagle3_acceptance_length (lines 228-314)
```python
def test_eagle3_acceptance_length(
    model_config: Eagle3ModelConfig,
    num_spec_tokens: int,
    tp_size: int,
    attention_backend: str,
    monkeypatch: pytest.MonkeyPatch,
):
    # Skip if this backend is incompatible with the model
    backend_enum = AttentionBackendEnum[attention_backend]
    if backend_enum in model_config.excluded_backends:
        pytest.skip(f"{attention_backend} is incompatible with {model_config.id}")

    with monkeypatch.context() as m:
        m.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
        with VllmRunner(
            model_name=model_config.verifier,
            speculative_config={
    # ... excerpt omitted for brevity ...
            expected = model_config.expected_acceptance_length
            expected_per_pos = model_config.expected_acceptance_lengths_per_pos
            rel_error = abs(actual_acceptance_length - expected) / expected
            assert rel_error <= DEFAULT_RTOL, (
                f"  Expected: {expected:.3f}\n"
            if expected_per_pos and len(expected_per_pos) == len(actual_per_pos):
                f"acceptance_length={actual_acceptance_length:.3f}"
                f" (expected={expected:.3f}, rel_error={rel_error:.2%})"
            )
            print(f"  Per-position: {[f'{v:.3f}' for v in actual_per_pos]}")
            if expected_per_pos:
                print(f"  Expected:     {[f'{v:.3f}' for v in expected_per_pos]}")
```
**EN:** Parameterized test covering `eagle3 acceptance length`. Parameter axes: `model_config, num_spec_tokens, tp_size, attention_backend`. Inputs/fixtures: `model_config, num_spec_tokens, tp_size, attention_backend, monkeypatch`. It exercises `large_gpu_mark, mark.skipif, mark.parametrize, get_tp_size_params, get_attention_backend_params, pytest.skip`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle3 acceptance length` 的测试用例。 参数维度：`model_config, num_spec_tokens, tp_size, attention_backend`。 输入或 fixture：`model_config, num_spec_tokens, tp_size, attention_backend, monkeypatch`。 该测试会调用 `large_gpu_mark, mark.skipif, mark.parametrize, get_tp_size_params, get_attention_backend_params, pytest.skip`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm, vllm.benchmarks.datasets, vllm.inputs, vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.attention.selector, vllm.v1.metrics.reader`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.benchmarks.datasets, vllm.inputs, vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.attention.selector, vllm.v1.metrics.reader`。
- **EN:** Local test helpers: `tests.conftest, tests.utils`.
- **CN:** 本地测试辅助模块：`tests.conftest, tests.utils`。
- **EN:** Standard-library support: `dataclasses, types`.
- **CN:** 标准库支持：`dataclasses, types`。
