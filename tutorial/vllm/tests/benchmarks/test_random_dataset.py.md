# test_random_dataset.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/benchmarks/test_random_dataset.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Random Dataset behavior in the Benchmarks test area through focused pytest scenarios. It focuses on scenarios such as HF Tokenizer, Params, Random Dataset Params. / 该文件在 Benchmarks 测试域中，通过有针对性的 pytest 场景验证 Random Dataset 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-14)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import random
from typing import Any, NamedTuple, cast

import numpy as np
import pytest
from transformers import AutoTokenizer, PreTrainedTokenizerBase

from vllm.benchmarks.datasets import (
    RandomDataset,
    RandomMultiModalDataset,
    SampleRequest,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `random`, `typing`, `numpy`, `pytest`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: hf_tokenizer (lines 17-20)
```python
@pytest.fixture(scope="session")
def hf_tokenizer() -> PreTrainedTokenizerBase:
    # Use a small, commonly available tokenizer
    return AutoTokenizer.from_pretrained("gpt2")
```
**EN:** Provides a pytest fixture for HF Tokenizer. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `AutoTokenizer.from_pretrained`.
**CN:** 该代码块定义 pytest 夹具 `hf_tokenizer`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `AutoTokenizer.from_pretrained` 构造或返回测试所需的值。

### Class: Params (lines 23-28)
```python
class Params(NamedTuple):
    num_requests: int
    prefix_len: int
    range_ratio: float
    input_len: int
    output_len: int
```
**EN:** Groups related scenarios for Params.
**CN:** 该类把与 Params 相关的场景组织在一起。

### Fixture: random_dataset_params (lines 31-35)
```python
@pytest.fixture(scope="session")
def random_dataset_params() -> Params:
    return Params(
        num_requests=16, prefix_len=7, range_ratio=0.3, input_len=50, output_len=20
    )
```
**EN:** Provides a pytest fixture for Random Dataset Params. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `Params`.
**CN:** 该代码块定义 pytest 夹具 `random_dataset_params`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `Params` 构造或返回测试所需的值。

### Helper: _fingerprint_sample (lines 38-40)
```python
def _fingerprint_sample(req: SampleRequest) -> tuple[str, int, int]:
    """Project a SampleRequest into a comparable tuple."""
    return (req.prompt, req.prompt_len, req.expected_output_len)
```
**EN:** Project a SampleRequest into a comparable tuple.
**CN:** 该辅助函数为 Fingerprint Sample 提供可复用逻辑，用于减少相关测试之间的重复代码。

### Helper: _collect_samples (lines 43-60)
```python
def _collect_samples(
    dataset: RandomDataset,
    tokenizer: PreTrainedTokenizerBase,
    num_requests: int = 16,
    prefix_len: int = 7,
    range_ratio: float = 0.3,
    input_len: int = 50,
    output_len: int = 20,
) -> list[tuple[str, int, int]]:
    samples = dataset.sample(
        tokenizer=tokenizer,
        num_requests=num_requests,
        prefix_len=prefix_len,
        range_ratio=range_ratio,
        input_len=input_len,
        output_len=output_len,
    )
    return [_fingerprint_sample(s) for s in samples]
```
**EN:** Implements a reusable helper for Collect Samples, reducing duplication across related tests. It coordinates operations such as `dataset.sample`, `_fingerprint_sample`.
**CN:** 该辅助函数为 Collect Samples 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `dataset.sample`, `_fingerprint_sample` 等操作。

### Test: test_random_dataset_same_seed (lines 63-101)
```python
@pytest.mark.benchmark
def test_random_dataset_same_seed(
    hf_tokenizer: PreTrainedTokenizerBase, random_dataset_params: Params
) -> None:
    """Same seed should yield identical outputs, even if global RNGs change.

    This guards against accidental reliance on Python's random or np.random
    in RandomDataset after moving to numpy.default_rng.
    """
    p = random_dataset_params
    common_seed = 123
    dataset_a = RandomDataset(random_seed=common_seed)
    dataset_b = RandomDataset(random_seed=common_seed)
    a = _collect_samples(
        dataset_a,
        hf_tokenizer,
        num_requests=p.num_requests,
        prefix_len=p.prefix_len,
        range_ratio=p.range_ratio,
# ... omitted for brevity ...
    _ = [random.random() for _ in range(100)]
    np.random.seed(888)
    _ = [np.random.random() for _ in range(100)]

    b = _collect_samples(
        dataset_b,
        hf_tokenizer,
        num_requests=p.num_requests,
        prefix_len=p.prefix_len,
        range_ratio=p.range_ratio,
        input_len=p.input_len,
        output_len=p.output_len,
    )
    assert a == b
```
**EN:** Same seed should yield identical outputs, even if global RNGs change. The body exercises logic via `RandomDataset`, `_collect_samples`, `random.seed` before asserting the expected outcome.
**CN:** 该测试用例验证 Random Dataset Same Seed 在特定场景下的行为。 函数体会先通过 `RandomDataset`, `_collect_samples`, `random.seed` 驱动目标逻辑，再断言预期结果。

### Test: test_random_dataset_different_seeds (lines 104-136)
```python
@pytest.mark.benchmark
def test_random_dataset_different_seeds(
    hf_tokenizer: PreTrainedTokenizerBase, random_dataset_params: Params
) -> None:
    """Different seeds should change outputs with overwhelming likelihood."""
    p = random_dataset_params
    seed_a = 0
    dataset_a = RandomDataset(random_seed=seed_a)
    a = _collect_samples(
        dataset_a,
        hf_tokenizer,
        num_requests=p.num_requests,
        prefix_len=p.prefix_len,
        range_ratio=p.range_ratio,
        input_len=p.input_len,
        output_len=p.output_len,
    )

    seed_b = 999
    dataset_b = RandomDataset(random_seed=seed_b)
    # Perturb global RNG with same seed as dataset_a to ensure isolation
    random.seed(seed_a)
    np.random.seed(seed_a)
    b = _collect_samples(
        dataset_b,
        hf_tokenizer,
        num_requests=p.num_requests,
        prefix_len=p.prefix_len,
        range_ratio=p.range_ratio,
        input_len=p.input_len,
        output_len=p.output_len,
    )
    assert a != b
```
**EN:** Different seeds should change outputs with overwhelming likelihood. The body exercises logic via `RandomDataset`, `_collect_samples`, `random.seed` before asserting the expected outcome.
**CN:** 该测试用例验证 Random Dataset Different Seeds 在特定场景下的行为。 函数体会先通过 `RandomDataset`, `_collect_samples`, `random.seed` 驱动目标逻辑，再断言预期结果。

### Test: test_random_mm_same_seed (lines 211-220)
```python
@pytest.mark.benchmark
def test_random_mm_same_seed(hf_tokenizer: PreTrainedTokenizerBase) -> None:
    seed = 42
    ds_a = RandomMultiModalDataset(random_seed=seed)
    ds_b = RandomMultiModalDataset(random_seed=seed)
    a = _collect_mm_samples(ds_a, hf_tokenizer)
    b = _collect_mm_samples(ds_b, hf_tokenizer)
    fa = [_mm_fingerprint_sample(s) for s in a]
    fb = [_mm_fingerprint_sample(s) for s in b]
    assert fa == fb
```
**EN:** Checks Random Mm Same Seed under a focused test scenario. The body exercises logic via `RandomMultiModalDataset`, `_collect_mm_samples`, `_mm_fingerprint_sample` before asserting the expected outcome.
**CN:** 该测试用例验证 Random Mm Same Seed 在特定场景下的行为。 函数体会先通过 `RandomMultiModalDataset`, `_collect_mm_samples`, `_mm_fingerprint_sample` 驱动目标逻辑，再断言预期结果。

### Test: test_random_mm_different_seeds (lines 223-233)
```python
@pytest.mark.benchmark
def test_random_mm_different_seeds(
    hf_tokenizer: PreTrainedTokenizerBase,
) -> None:
    ds_a = RandomMultiModalDataset(random_seed=0)
    ds_b = RandomMultiModalDataset(random_seed=999)
    a = _collect_mm_samples(ds_a, hf_tokenizer)
    b = _collect_mm_samples(ds_b, hf_tokenizer)
    fa = [_mm_fingerprint_sample(s) for s in a]
    fb = [_mm_fingerprint_sample(s) for s in b]
    assert fa != fb
```
**EN:** Checks Random Mm Different Seeds under a focused test scenario. The body exercises logic via `RandomMultiModalDataset`, `_collect_mm_samples`, `_mm_fingerprint_sample` before asserting the expected outcome.
**CN:** 该测试用例验证 Random Mm Different Seeds 在特定场景下的行为。 函数体会先通过 `RandomMultiModalDataset`, `_collect_mm_samples`, `_mm_fingerprint_sample` 驱动目标逻辑，再断言预期结果。

### Test: test_random_mm_respects_limits (lines 236-252)
```python
@pytest.mark.benchmark
def test_random_mm_respects_limits(
    hf_tokenizer: PreTrainedTokenizerBase,
) -> None:
    ds = RandomMultiModalDataset(random_seed=0)
    # Requesting 3 items with a per-prompt limit of 1 should error per current
    # design (dataset refuses to silently clamp below the requested baseline).
    with pytest.raises(ValueError):
        _collect_mm_samples(
            ds,
            hf_tokenizer,
            num_requests=12,
            base_items_per_request=3,
            num_mm_items_range_ratio=0.0,
            limit_mm_per_prompt={"image": 1, "video": 0},
            bucket_config={(32, 32, 1): 1.0},
        )
```
**EN:** Checks Random Mm Respects Limits under a focused test scenario. The body exercises logic via `RandomMultiModalDataset`, `pytest.raises`, `_collect_mm_samples` before asserting the expected outcome.
**CN:** 该测试用例验证 Random Mm Respects Limits 在特定场景下的行为。 函数体会先通过 `RandomMultiModalDataset`, `pytest.raises`, `_collect_mm_samples` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
_mm_fingerprint_sample
_collect_mm_samples
test_random_mm_zero_prob_entries_are_removed
test_random_mm_zero_items
test_random_mm_num_items_per_prompt
test_random_mm_bucket_config_not_mutated
test_random_mm_video_sampling
test_random_mm_video_only_sampling
test_random_mm_video_deterministic_sampling
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。
- **Benchmarking / 基准测试**
  - **EN:** The tests validate CLI arguments, datasets, or metrics used by benchmark workflows.
  - **CN:** 测试验证基准流程使用的命令行参数、数据集或指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `typing`
- **Third-party / 第三方依赖**: `numpy`, `pytest`, `transformers`
- **vLLM internal / vLLM 内部依赖**: `vllm.benchmarks.datasets`
