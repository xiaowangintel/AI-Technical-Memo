# test_rejection_sampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/sample/test_rejection_sampler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `rejection sampler` behavior and regressions in the v1 stack. / 验证 v1 栈中 `rejection sampler` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-20)
```python
from typing import Any
from unittest.mock import Mock

import pytest
import torch
import torch.nn.functional as F

from tests.v1.sample.utils import create_allowed_token_ids
from vllm.platforms import current_platform
from vllm.v1.sample.logits_processor import LogitsProcessors
from vllm.v1.sample.metadata import SamplingMetadata
from vllm.v1.sample.rejection_sampler import (
    PLACEHOLDER_TOKEN_ID,
    RejectionSampler,
    sample_recovered_tokens,
)
from vllm.v1.sample.sampler import Sampler, SamplerOutput
from vllm.v1.spec_decode.metadata import SpecDecodeMetadata
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, torch.nn.functional`. vLLM modules under test include `vllm.platforms, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.rejection_sampler, vllm.v1.sample.sampler, ...`. Local helpers come from `tests.v1.sample.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, torch.nn.functional`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.rejection_sampler, vllm.v1.sample.sampler, ...`。 本地测试辅助逻辑来自 `tests.v1.sample.utils`。

### Module state / 模块级状态 (line 22)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DEVICE_TYPE`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DEVICE_TYPE`。

### rejection_sampler (lines 26-29)
```python
def rejection_sampler():
    mock_sampler = Mock(spec=Sampler)
    mock_sampler.logprobs_mode = "raw_logprobs"
    return RejectionSampler(mock_sampler)
```
**EN:** Fixture/helper `rejection_sampler` prepares reusable state for downstream tests. Key calls include `Mock, RejectionSampler`.
**CN:** `rejection_sampler` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `Mock, RejectionSampler`。

### mock_sampler_output (lines 32-37)
```python
def mock_sampler_output(
    rejection_sampler: RejectionSampler, bonus_token_ids: torch.Tensor
):
    rejection_sampler.sampler.return_value = SamplerOutput(
        sampled_token_ids=bonus_token_ids, logprobs_tensors=None
    )
```
**EN:** Helper function `mock_sampler_output` encapsulates reusable logic for `sampler output`. Inputs: `rejection_sampler, bonus_token_ids`. Key calls include `SamplerOutput`.
**CN:** 辅助函数 `mock_sampler_output` 封装了与 `sampler output` 相关的可复用逻辑。 输入参数：`rejection_sampler, bonus_token_ids`。 关键调用包括 `SamplerOutput`。

### create_spec_decode_metadata (lines 40-48)
```python
def create_spec_decode_metadata(
    spec_tokens: list[list[int]], logits: torch.Tensor
) -> SpecDecodeMetadata:
    metadata = SpecDecodeMetadata.make_dummy(spec_tokens, device=logits.device)
    metadata.target_logits_indices = torch.arange(logits.shape[0])
    # Output bonus token ids are mocked, so the bonus logit indices should
    # be empty.
    metadata.bonus_logits_indices = torch.empty(0, dtype=torch.int32)
    return metadata
```
**EN:** Helper function `create_spec_decode_metadata` encapsulates reusable logic for `spec decode metadata`. Inputs: `spec_tokens, logits`. Key calls include `SpecDecodeMetadata.make_dummy, torch.arange, torch.empty`.
**CN:** 辅助函数 `create_spec_decode_metadata` 封装了与 `推测解码 metadata` 相关的可复用逻辑。 输入参数：`spec_tokens, logits`。 关键调用包括 `SpecDecodeMetadata.make_dummy, torch.arange, torch.empty`。

### create_logits_tensor (lines 51-68)
```python
def create_logits_tensor(
    output_token_ids: list[list[int]],
    vocab_size: int = 100,
    token_idx_to_override: int | None = None,
) -> torch.Tensor:
    """Helper function to create logits tensor that
    will produce desired token ids on argmax"""
    token_ids = [tokens[:-1] for tokens in output_token_ids]
    num_total_tokens = sum(len(tokens) for tokens in token_ids)
    logits = torch.full((num_total_tokens, vocab_size), -100.0, device=DEVICE_TYPE)
    start_loc = 0
    for tokens in token_ids:
        for j, token_id in enumerate(tokens):
            logits[start_loc + j, token_id] = 100.0
        start_loc += len(tokens)
    if token_idx_to_override:
        logits[:, token_idx_to_override] = 99.0
    return logits
```
**EN:** Helper function `create_logits_tensor` encapsulates reusable logic for `logits tensor`. Inputs: `output_token_ids, vocab_size, token_idx_to_override`. Key calls include `sum, torch.full, enumerate, len`.
**CN:** 辅助函数 `create_logits_tensor` 封装了与 `logits tensor` 相关的可复用逻辑。 输入参数：`output_token_ids, vocab_size, token_idx_to_override`。 关键调用包括 `sum, torch.full, enumerate, len`。

### create_sampling_metadata (lines 71-129)
```python
def create_sampling_metadata(
    all_greedy: bool,
    output_token_ids: list[list[int]] | None = None,
    prompt_token_ids: torch.Tensor | None = None,
    spec_token_ids: torch.Tensor | None = None,
    temperature: torch.Tensor | None = None,
    top_k: torch.Tensor | None = None,
    top_p: torch.Tensor | None = None,
    generators: dict[int, Any] | None = None,
    frequency_penalties: list[float] | None = None,
    presence_penalties: list[float] | None = None,
    repetition_penalties: list[float] | None = None,
    bad_words_token_ids: dict[int, list[list[int]]] | None = None,
    allowed_token_ids_mask: torch.Tensor | None = None,
) -> SamplingMetadata:
    """Create a v1 sampling metadata object with all_greedy set
    to the given value. Either all greedy or all random sampling
    is used.
    # ... excerpt omitted for brevity ...
        assert temperature is not None
        assert output_token_ids
        assert len(output_token_ids) > 0
    return SamplingMetadata(
        output_token_ids=[] if output_token_ids is None else output_token_ids,
        spec_token_ids=[] if spec_token_ids is None else spec_token_ids,
        allowed_token_ids_mask=allowed_token_ids_mask,
        bad_words_token_ids={} if bad_words_token_ids is None else bad_words_token_ids,
        logitsprocs=LogitsProcessors(),
    )
```
**EN:** Helper function `create_sampling_metadata` encapsulates reusable logic for `sampling metadata`. Inputs: `all_greedy, output_token_ids, prompt_token_ids, spec_token_ids, temperature, top_k, top_p, generators, ...`. Key calls include `any, SamplingMetadata, torch.tensor, len, LogitsProcessors`. It includes 3 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `create_sampling_metadata` 封装了与 `采样 metadata` 相关的可复用逻辑。 输入参数：`all_greedy, output_token_ids, prompt_token_ids, spec_token_ids, temperature, top_k, top_p, generators, ...`。 关键调用包括 `any, SamplingMetadata, torch.tensor, len, LogitsProcessors`。 其中包含 3 个内部断言，用于保护前置假设。

### test_perfect_match (lines 133-151)
```python
def test_perfect_match(rejection_sampler):
    """Test when output tokens perfectly match speculated tokens"""
    spec_tokens = [[1, 2, 3]]
    output_tokens = [[1, 2, 3, 4]]  # 4 is the bonus token

    metadata = create_sampling_metadata(all_greedy=True)
    logits = create_logits_tensor(output_tokens)
    bonus_token_tensor = torch.tensor([output_tokens[0][-1]], device=logits.device)
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(rejection_sampler, bonus_token_tensor)
    output = rejection_sampler(
        spec_decode_metadata,
        draft_probs=None,
        logits=logits,
        sampling_metadata=metadata,
    )
    expected = torch.tensor([[1, 2, 3, 4]], dtype=torch.int, device=logits.device)
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `perfect match`. Inputs/fixtures: `rejection_sampler`. It exercises `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `perfect match` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`。 代码主体包含 1 个显式断言。

### test_early_mismatch (lines 154-176)
```python
def test_early_mismatch(rejection_sampler):
    """Test when there's an early mismatch in tokens"""
    spec_tokens = [[1, 2, 3]]
    output_tokens = [[1, 5, 3, 4]]  # Mismatch at position 1

    metadata = create_sampling_metadata(all_greedy=True)
    logits = create_logits_tensor(output_tokens)
    bonus_token_tensor = torch.tensor([output_tokens[0][-1]], device=logits.device)
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(rejection_sampler, bonus_token_tensor)
    output = rejection_sampler(
        spec_decode_metadata,
        draft_probs=None,
        logits=logits,
        sampling_metadata=metadata,
    )
    expected = torch.tensor(
        [[1, 5, PLACEHOLDER_TOKEN_ID, PLACEHOLDER_TOKEN_ID]],
        dtype=torch.int,
        device=logits.device,
    )
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `early mismatch`. Inputs/fixtures: `rejection_sampler`. It exercises `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `early mismatch` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`。 代码主体包含 1 个显式断言。

### test_multiple_sequences (lines 179-201)
```python
def test_multiple_sequences(rejection_sampler):
    """Test handling multiple sequences of speculated tokens"""
    spec_tokens = [[1, 2], [3]]
    output_tokens = [[1, 2, 5], [3, 4]]  # Two sequences with bonus tokens 5 and 4

    metadata = create_sampling_metadata(all_greedy=True)
    logits = create_logits_tensor(output_tokens)
    bonus_token_tensor = torch.tensor(
        [output_tokens[0][-1], output_tokens[1][-1]], device=logits.device
    )
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(rejection_sampler, bonus_token_tensor)
    output = rejection_sampler(
        spec_decode_metadata,
        draft_probs=None,
        logits=logits,
        sampling_metadata=metadata,
    )
    expected = torch.tensor(
        [[1, 2, 5], [3, 4, PLACEHOLDER_TOKEN_ID]], dtype=torch.int, device=logits.device
    )
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `multiple sequences`. Inputs/fixtures: `rejection_sampler`. It exercises `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `multiple sequences` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`。 代码主体包含 1 个显式断言。

### test_single_token_sequence (lines 204-222)
```python
def test_single_token_sequence(rejection_sampler):
    """Test handling sequences with single token"""
    spec_tokens = [[1]]
    output_tokens = [[1, 2]]  # Single token with bonus token 2

    metadata = create_sampling_metadata(all_greedy=True)
    logits = create_logits_tensor(output_tokens)
    bonus_token_tensor = torch.tensor([output_tokens[0][-1]], device=logits.device)
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(rejection_sampler, bonus_token_tensor)
    output = rejection_sampler(
        spec_decode_metadata,
        draft_probs=None,
        logits=logits,
        sampling_metadata=metadata,
    )
    expected = torch.tensor([[1, 2]], dtype=torch.int, device=logits.device)
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `single token sequence`. Inputs/fixtures: `rejection_sampler`. It exercises `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `single token sequence` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`。 代码主体包含 1 个显式断言。

### test_empty_sequence (lines 225-243)
```python
def test_empty_sequence(rejection_sampler):
    """Test handling empty sequence of speculated tokens"""
    spec_tokens: list[list[int]] = [[]]
    output_tokens = [[5]]  # Just the bonus token

    metadata = create_sampling_metadata(all_greedy=True)
    logits = create_logits_tensor(output_tokens)
    bonus_token_tensor = torch.tensor([output_tokens[0][-1]], device=logits.device)
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(rejection_sampler, bonus_token_tensor)
    output = rejection_sampler(
        spec_decode_metadata,
        draft_probs=None,
        logits=logits,
        sampling_metadata=metadata,
    )
    expected = torch.tensor([[5]], dtype=torch.int, device=logits.device)
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `empty sequence`. Inputs/fixtures: `rejection_sampler`. It exercises `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `empty sequence` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`。 代码主体包含 1 个显式断言。

### test_multiple_mismatches (lines 246-273)
```python
def test_multiple_mismatches(rejection_sampler):
    """Test handling multiple sequences with mismatches"""
    spec_tokens = [[1, 2, 3], [4, 5, 6]]
    output_tokens = [[1, 2, 7, 6], [4, 8, 6, 9]]  # Mismatches in both sequences

    metadata = create_sampling_metadata(all_greedy=True)
    logits = create_logits_tensor(output_tokens)
    bonus_token_tensor = torch.tensor(
        [output_tokens[0][-1], output_tokens[1][-1]], device=logits.device
    )
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(rejection_sampler, bonus_token_tensor)
    output = rejection_sampler(
        spec_decode_metadata,
        draft_probs=None,
        logits=logits,
        sampling_metadata=metadata,
    )
    expected = torch.tensor(
        [
            [1, 2, 7, PLACEHOLDER_TOKEN_ID],
            [4, 8, PLACEHOLDER_TOKEN_ID, PLACEHOLDER_TOKEN_ID],
        ],
        dtype=torch.int,
        device=logits.device,
    )
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `multiple mismatches`. Inputs/fixtures: `rejection_sampler`. It exercises `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `multiple mismatches` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`。 代码主体包含 1 个显式断言。

### test_parametrized_cases (lines 288-305)
```python
def test_parametrized_cases(rejection_sampler, spec_tokens, output_tokens, expected):
    """Parametrized test for various matching scenarios"""
    metadata = create_sampling_metadata(all_greedy=True)
    logits = create_logits_tensor(output_tokens)
    bonus_token_tensor = torch.tensor(
        [tokens[-1] for tokens in output_tokens], device=logits.device
    )
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(rejection_sampler, bonus_token_tensor)
    output = rejection_sampler(
        spec_decode_metadata,
        draft_probs=None,
        logits=logits,
        sampling_metadata=metadata,
    )
    expected_tensor = torch.tensor(expected, dtype=torch.int, device=logits.device)
    assert torch.equal(output.sampled_token_ids, expected_tensor)
```
**EN:** Parameterized test covering `parametrized cases`. Parameter axes: `spec_tokens, output_tokens, expected`. Inputs/fixtures: `rejection_sampler, spec_tokens, output_tokens, expected`. It exercises `mark.parametrize, create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `parametrized cases` 的测试用例。 参数维度：`spec_tokens, output_tokens, expected`。 输入或 fixture：`rejection_sampler, spec_tokens, output_tokens, expected`。 该测试会调用 `mark.parametrize, create_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata, mock_sampler_output`。 代码主体包含 1 个显式断言。

### test_deterministic_when_seeded (lines 314-377)
```python
def test_deterministic_when_seeded(
    rejection_sampler,
    k: int,
    vocab_size: int,
    batch_size: int,
    frac_seeded: float,
    n_rep: int,
):
    num_tokens = batch_size * k
    draft_probs = torch.rand(
        num_tokens,
        vocab_size,
        dtype=torch.float32,
        device=DEVICE_TYPE,
    )
    draft_probs = F.softmax(draft_probs, dim=-1)
    target_logits = torch.rand_like(draft_probs)
    bonus_token_ids = torch.randint(
    # ... excerpt omitted for brevity ...
        results.append(rep_result.sampled_token_ids)

    for i in range(batch_size):
        if seeded_mask[i]:
            for j in range(1, n_rep):
                assert torch.equal(results[j][i], results[0][i])
```
**EN:** Parameterized test covering `deterministic when seeded`. Parameter axes: `k, vocab_size, batch_size, frac_seeded, n_rep`. Inputs/fixtures: `rejection_sampler, k, vocab_size, batch_size, frac_seeded, n_rep`. It exercises `mark.parametrize, torch.rand, F.softmax, torch.rand_like, torch.randint, range`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `deterministic when seeded` 的测试用例。 参数维度：`k, vocab_size, batch_size, frac_seeded, n_rep`。 输入或 fixture：`rejection_sampler, k, vocab_size, batch_size, frac_seeded, n_rep`。 该测试会调用 `mark.parametrize, torch.rand, F.softmax, torch.rand_like, torch.randint, range`。 代码主体包含 1 个显式断言。

### test_rejection_sampling_approximates_target_distribution (lines 380-465)
```python
def test_rejection_sampling_approximates_target_distribution():
    """Verify rejection sampling approximates target distribution,
    despite sampling from a potentially distinct draft distribution.

    This is done by first creating a random target probability
    distribution and a random draft probability distribution. We then
    sample token ids from the rejection sampler using these draft
    and target distributions. The samples are used to estimate
    the output probability distribution, which we expect to approximate
    the target distribution.
    A basic distance metric is used to determine similarity between
    distributions.
    We expect that as we increase the number of samples,
    the distance between the observed distribution and the target
    distribution decreases. To measure this, we compare the distance
    of the observed distribution against both the target distribution
    # ... excerpt omitted for brevity ...
    expected_improvement_multiplier = 20
    assert (
        relative_change_in_distance_wrt_target
        > relative_change_in_distance_wrt_reference * expected_improvement_multiplier
    )
```
**EN:** Test case covering `rejection sampling approximates target distribution`. It exercises `torch.set_default_device, F.softmax, torch.rand, get_ratio_first_to_last, estimate_rejection_sampling_pdf, rej_sample_probs.to`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `rejection 采样 approximates target distribution` 的测试用例。 该测试会调用 `torch.set_default_device, F.softmax, torch.rand, get_ratio_first_to_last, estimate_rejection_sampling_pdf, rej_sample_probs.to`。 代码主体包含 1 个显式断言。

### get_ratio_first_to_last (lines 468-469)
```python
def get_ratio_first_to_last(elements: list[float]) -> float:
    return elements[0] / elements[-1]
```
**EN:** Helper function `get_ratio_first_to_last` encapsulates reusable logic for `ratio first to last`. Inputs: `elements`.
**CN:** 辅助函数 `get_ratio_first_to_last` 封装了与 `ratio first to last` 相关的可复用逻辑。 输入参数：`elements`。

### estimate_rejection_sampling_pdf (lines 472-535)
```python
def estimate_rejection_sampling_pdf(
    draft_probs: torch.Tensor,
    target_logits: torch.Tensor,
    k: int,
    vocab_size: int,
    num_samples: int,
) -> torch.Tensor:
    """Estimate the probability distribution of the output tokens
    using rejection sampling.

    Args:
        draft_probs: Draft probability distribution.
        target_logits: Target logits.
        num_samples: Number of samples to draw.
    Returns:
        Estimated probability distribution of the output tokens.
    """
    # ... excerpt omitted for brevity ...
    mock_sampler = Mock(spec=Sampler)
        bins=vocab_size,
        range=(0, vocab_size),
        density=True,
    )
    return hist.hist
```
**EN:** Helper function `estimate_rejection_sampling_pdf` encapsulates reusable logic for `estimate rejection sampling pdf`. Inputs: `draft_probs, target_logits, k, vocab_size, num_samples`. Key calls include `Mock, RejectionSampler, reshape.repeat, multinomial.reshape, draft_probs.view, zeros.repeat`.
**CN:** 辅助函数 `estimate_rejection_sampling_pdf` 封装了与 `estimate rejection 采样 pdf` 相关的可复用逻辑。 输入参数：`draft_probs, target_logits, k, vocab_size, num_samples`。 关键调用包括 `Mock, RejectionSampler, reshape.repeat, multinomial.reshape, draft_probs.view, zeros.repeat`。

### native_sample_recovered_tokens (lines 538-599)
```python
def native_sample_recovered_tokens(
    max_spec_len: int,
    num_draft_tokens: list[int],
    cu_num_draft_tokens: torch.Tensor,  # [batch_size]
    draft_token_ids: torch.Tensor,  # [num_tokens]
    draft_probs: torch.Tensor | None,  # [num_tokens, vocab_size]
    target_probs: torch.Tensor,  # [num_tokens, vocab_size]
    sampling_metadata: SamplingMetadata,
    device: torch.device,
) -> torch.Tensor:
    batch_size = len(num_draft_tokens)
    vocab_size = target_probs.shape[-1]

    q = torch.empty(
        (batch_size, vocab_size),
        dtype=torch.float32,
        device=device,
    )
    # ... excerpt omitted for brevity ...
                )
            score = prob * inv_q[req_idx]
            recovered_id = torch.argmax(score, dim=-1)
            out[token_idx] = recovered_id
    return out
```
**EN:** Helper function `native_sample_recovered_tokens` encapsulates reusable logic for `native sample recovered tokens`. Inputs: `max_spec_len, num_draft_tokens, cu_num_draft_tokens, draft_token_ids, draft_probs, target_probs, sampling_metadata, device`. Key calls include `len, torch.empty, q.exponential_, generators.items, q.reciprocal, torch.empty_like`.
**CN:** 辅助函数 `native_sample_recovered_tokens` 封装了与 `native sample recovered tokens` 相关的可复用逻辑。 输入参数：`max_spec_len, num_draft_tokens, cu_num_draft_tokens, draft_token_ids, draft_probs, target_probs, sampling_metadata, device`。 关键调用包括 `len, torch.empty, q.exponential_, generators.items, q.reciprocal, torch.empty_like`。

### _test_masked_logits (lines 602-652)
```python
def _test_masked_logits(
    rejection_sampler,
    batch_size: int,
    num_draft_tokens: int,
    vocab_size: int,
    target_logits: torch.Tensor,
    unmasked_indices: torch.Tensor,
    sampling_metadata: SamplingMetadata,
):
    # Set up test parameters
    num_tokens = batch_size * num_draft_tokens

    # Create random draft probabilities.
    draft_probs = torch.rand(
        (num_tokens, vocab_size), dtype=torch.float32, device=DEVICE_TYPE
    )
    draft_probs = F.softmax(draft_probs, dim=-1)
    # ... excerpt omitted for brevity ...
    # Check that all sampled tokens are within the unmasked indices.
    for i in range(num_tokens):
        token_id = output_token_ids[i]
        if token_id == PLACEHOLDER_TOKEN_ID:
            continue
        assert token_id in unmasked_indices[i]
```
**EN:** Helper function `_test_masked_logits` encapsulates reusable logic for `masked logits`. Inputs: `rejection_sampler, batch_size, num_draft_tokens, vocab_size, target_logits, unmasked_indices, sampling_metadata`. Key calls include `torch.rand, F.softmax, torch.multinomial, draft_token_ids.reshape, draft_token_ids.tolist, torch.zeros`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_test_masked_logits` 封装了与 `masked logits` 相关的可复用逻辑。 输入参数：`rejection_sampler, batch_size, num_draft_tokens, vocab_size, target_logits, unmasked_indices, sampling_metadata`。 关键调用包括 `torch.rand, F.softmax, torch.multinomial, draft_token_ids.reshape, draft_token_ids.tolist, torch.zeros`。 其中包含 1 个内部断言，用于保护前置假设。

### test_top_k (lines 656-695)
```python
def test_top_k(rejection_sampler, top_k):
    """Test rejection sampling with top-k sampling"""
    vocab_size = 100
    batch_size = 100
    num_draft_tokens = 3
    num_tokens = batch_size * num_draft_tokens

    # Randomly create top-k indices.
    top_k_indices = [
        torch.randperm(vocab_size, device=DEVICE_TYPE)[:top_k]
        for _ in range(num_tokens)
    ]
    top_k_indices = torch.stack(top_k_indices)
    # Create logits with the uniform distribution.
    target_logits = torch.zeros((num_tokens, vocab_size), device=DEVICE_TYPE)
    # Increment the logits for top-k indices, a little bit more than the other
    # ... excerpt omitted for brevity ...
        num_draft_tokens=num_draft_tokens,
        vocab_size=vocab_size,
        target_logits=target_logits,
        unmasked_indices=top_k_indices,
        sampling_metadata=sampling_metadata,
    )
```
**EN:** Parameterized test covering `top k`. Parameter axes: `top_k`. Inputs/fixtures: `rejection_sampler, top_k`. It exercises `mark.parametrize, torch.stack, torch.zeros, range, torch.ones, create_sampling_metadata`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `top k` 的测试用例。 参数维度：`top_k`。 输入或 fixture：`rejection_sampler, top_k`。 该测试会调用 `mark.parametrize, torch.stack, torch.zeros, range, torch.ones, create_sampling_metadata`。 主要通过 mock、回调或输出检查来完成验证。

### test_top_p (lines 699-742)
```python
def test_top_p(rejection_sampler, top_p):
    """Test rejection sampling with top-p sampling"""
    vocab_size = 100
    batch_size = 100
    num_draft_tokens = 3
    num_tokens = batch_size * num_draft_tokens

    # Create logits with the uniform distribution.
    target_logits = torch.randn((num_tokens, vocab_size), device=DEVICE_TYPE)
    temperature = torch.ones(batch_size, dtype=torch.float32, device=DEVICE_TYPE)
    rescaled_logits = target_logits / temperature
    logits_sort, logits_idx = rescaled_logits.sort(dim=-1, descending=False)
    probs_sort = logits_sort.softmax(dim=-1)
    probs_sum = probs_sort.cumsum(dim=-1)
    top_p_mask = probs_sum <= 1 - top_p
    # at least one
    top_p_mask[:, -1] = False
    # ... excerpt omitted for brevity ...
        num_draft_tokens=num_draft_tokens,
        vocab_size=vocab_size,
        target_logits=target_logits,
        unmasked_indices=top_p_indices,
        sampling_metadata=sampling_metadata,
    )
```
**EN:** Parameterized test covering `top p`. Parameter axes: `top_p`. Inputs/fixtures: `rejection_sampler, top_p`. It exercises `mark.parametrize, torch.randn, torch.ones, rescaled_logits.sort, logits_sort.softmax, probs_sort.cumsum`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `top p` 的测试用例。 参数维度：`top_p`。 输入或 fixture：`rejection_sampler, top_p`。 该测试会调用 `mark.parametrize, torch.randn, torch.ones, rescaled_logits.sort, logits_sort.softmax, probs_sort.cumsum`。 主要通过 mock、回调或输出检查来完成验证。

### test_frequency_penalties (lines 746-783)
```python
def test_frequency_penalties(rejection_sampler):
    """Test rejection sampling with frequency penalties"""
    spec_tokens = [[1, 1, 1], [], [1, 1, 1]]
    output_tokens = [[1, 1, 1, 1], [7], [1, 1, 1, 1]]  # 1, 7 and 1 are the bonus tokens

    num_requests = len(spec_tokens)
    logits = create_logits_tensor(output_tokens, token_idx_to_override=15)
    metadata = create_sampling_metadata(
        all_greedy=True,
        output_token_ids=[[2], [3], [4]],
        spec_token_ids=spec_tokens,
        prompt_token_ids=torch.tensor(
            [[5, 6, 7], [6, 7, 8], [7, 8, 9]],
            device=DEVICE_TYPE,
        ),
        frequency_penalties=[1.5, 1.5, 0.7],
        presence_penalties=[0.0] * num_requests,
        repetition_penalties=[1.0] * num_requests,
    # ... excerpt omitted for brevity ...
    expected = torch.tensor(
        [[1, 15, -1, -1], [7, -1, -1, -1], [1, 1, 15, -1]],
        dtype=torch.int,
        device=logits.device,
    )
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `frequency penalties`. Inputs/fixtures: `rejection_sampler`. It exercises `len, create_logits_tensor, create_sampling_metadata, torch.tensor, SpecDecodeMetadata.make_dummy, mock_sampler_output`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `frequency penalties` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `len, create_logits_tensor, create_sampling_metadata, torch.tensor, SpecDecodeMetadata.make_dummy, mock_sampler_output`。 代码主体包含 1 个显式断言。

### test_bad_words (lines 786-826)
```python
def test_bad_words(rejection_sampler):
    """Test rejection sampling with bad words constraints.

    This test applies bad words to non-consecutive requests (0 and 2, but not 1)
    to verify correct logit indexing when iterating over requests with bad words.
    """
    spec_tokens = [[1, 2, 3], [1, 15, 3], [1, 2, 3]]
    output_tokens = [[1, 2, 3, 4], [1, 15, 3, 4], [1, 2, 3, 4]]
    logits = create_logits_tensor(output_tokens, token_idx_to_override=15)
    metadata = create_sampling_metadata(
        all_greedy=True,
        output_token_ids=[[2], [3], [4]],
        spec_token_ids=spec_tokens,
        bad_words_token_ids={
            0: [[2]],
            # Request 1 has no bad words (to test non-consecutive request handling)
            2: [[2]],
    # ... excerpt omitted for brevity ...
    expected = torch.tensor(
        [[1, 15, -1, -1], [1, 15, 3, 4], [1, 15, -1, -1]],
        dtype=torch.int,
        device=logits.device,
    )
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `bad words`. Inputs/fixtures: `rejection_sampler`. It exercises `create_logits_tensor, create_sampling_metadata, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `bad words` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_logits_tensor, create_sampling_metadata, torch.tensor, create_spec_decode_metadata, mock_sampler_output, rejection_sampler`。 代码主体包含 1 个显式断言。

### test_allowed_token_ids (lines 829-873)
```python
def test_allowed_token_ids(rejection_sampler):
    """Test rejection sampling with allowed token ids"""
    spec_tokens = [[1, 2, 10], [10, 5, 3], [7, 10, 12]]
    output_tokens = [[1, 2, 10, 5], [10, 5, 10, 5], [7, 10, 12, 5]]
    # Not allowed tokens:
    # 0: 0-4
    # 1: 1-5
    # 2: 2-6
    num_allowed_token_ids = 5

    # Use the token 15 as the sampler choose if a token rejected
    logits = create_logits_tensor(output_tokens, token_idx_to_override=15)
    batch_size = len(output_tokens)
    _, vocab_size = logits.size()
    mask = create_allowed_token_ids(
        batch_size=batch_size,
        vocab_size=vocab_size,
    # ... excerpt omitted for brevity ...
    expected = torch.tensor(
        [[15, -1, -1, -1], [10, 5, 10, -1], [7, 10, 12, 5]],
        dtype=torch.int,
        device=logits.device,
    )
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Test case covering `allowed token ids`. Inputs/fixtures: `rejection_sampler`. It exercises `create_logits_tensor, len, logits.size, create_allowed_token_ids, create_sampling_metadata, torch.tensor`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `allowed token ids` 的测试用例。 输入或 fixture：`rejection_sampler`。 该测试会调用 `create_logits_tensor, len, logits.size, create_allowed_token_ids, create_sampling_metadata, torch.tensor`。 代码主体包含 1 个显式断言。

### test_sample_recovered_tokens (lines 880-935)
```python
def test_sample_recovered_tokens(
    batch_size: int, vocab_size: int, max_spec_len: int, no_draft_probs: bool
):
    num_tokens = batch_size * max_spec_len

    # Create random draft probabilities.
    draft_probs = torch.rand(
        num_tokens,
        vocab_size,
        dtype=torch.float32,
        device=DEVICE_TYPE,
    )
    draft_probs = F.softmax(draft_probs, dim=-1)
    # Create random target probabilities.
    target_logits = torch.rand(
        num_tokens, vocab_size, dtype=torch.float32, device=DEVICE_TYPE
    # ... excerpt omitted for brevity ...
        None if no_draft_probs else draft_probs,
        target_probs,
        sampling_metadata,
    assert torch.equal(recovered_token_ids, ref_recovered_token_ids)
```
**EN:** Parameterized test covering `sample recovered tokens`. Parameter axes: `batch_size, vocab_size, max_spec_len, no_draft_probs`. Inputs/fixtures: `batch_size, vocab_size, max_spec_len, no_draft_probs`. It exercises `mark.parametrize, torch.rand, F.softmax, multinomial.to, torch.ones, create_sampling_metadata`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `sample recovered tokens` 的测试用例。 参数维度：`batch_size, vocab_size, max_spec_len, no_draft_probs`。 输入或 fixture：`batch_size, vocab_size, max_spec_len, no_draft_probs`。 该测试会调用 `mark.parametrize, torch.rand, F.softmax, multinomial.to, torch.ones, create_sampling_metadata`。 代码主体包含 1 个显式断言。

### _make_synthetic_sampler (lines 941-947)
```python
def _make_synthetic_sampler(rates: list[float]) -> RejectionSampler:
    mock_sampler = Mock(spec=Sampler)
    mock_sampler.logprobs_mode = "raw_logprobs"
    spec_config = Mock()
    spec_config.rejection_sample_method = "synthetic"
    spec_config.synthetic_acceptance_rates = rates
    return RejectionSampler(mock_sampler, spec_config, torch.device(DEVICE_TYPE))
```
**EN:** Helper function `_make_synthetic_sampler` encapsulates reusable logic for `synthetic sampler`. Inputs: `rates`. Key calls include `Mock, RejectionSampler, torch.device`.
**CN:** 辅助函数 `_make_synthetic_sampler` 封装了与 `synthetic sampler` 相关的可复用逻辑。 输入参数：`rates`。 关键调用包括 `Mock, RejectionSampler, torch.device`。

### _make_sampling_metadata (lines 950-952)
```python
def _make_sampling_metadata(all_greedy: bool) -> SamplingMetadata:
    temperature = None if all_greedy else torch.tensor([1.0, 1.0], device=DEVICE_TYPE)
    return create_sampling_metadata(all_greedy=all_greedy, temperature=temperature)
```
**EN:** Helper function `_make_sampling_metadata` encapsulates reusable logic for `sampling metadata`. Inputs: `all_greedy`. Key calls include `create_sampling_metadata, torch.tensor`.
**CN:** 辅助函数 `_make_sampling_metadata` 封装了与 `采样 metadata` 相关的可复用逻辑。 输入参数：`all_greedy`。 关键调用包括 `create_sampling_metadata, torch.tensor`。

### test_synthetic_all_accepted (lines 956-974)
```python
def test_synthetic_all_accepted(all_greedy: bool):
    """With all rates=1.0, every draft token is accepted."""
    sampler = _make_synthetic_sampler([1.0, 1.0])
    spec_tokens = [[1, 2], [3]]
    output_tokens = [[10, 20, 50], [30, 40]]

    metadata = _make_sampling_metadata(all_greedy)
    logits = create_logits_tensor(output_tokens)
    bonus = torch.tensor([50, 40], device=DEVICE_TYPE)
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(sampler, bonus)
    output = sampler(spec_decode_metadata, None, logits, metadata)
    expected = torch.tensor(
        [[1, 2, 50], [3, 40, PLACEHOLDER_TOKEN_ID]],
        dtype=torch.int,
        device=DEVICE_TYPE,
    )
    assert torch.equal(output.sampled_token_ids, expected)
```
**EN:** Parameterized test covering `synthetic all accepted`. Parameter axes: `all_greedy`. Inputs/fixtures: `all_greedy`. It exercises `mark.parametrize, _make_synthetic_sampler, _make_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `synthetic all accepted` 的测试用例。 参数维度：`all_greedy`。 输入或 fixture：`all_greedy`。 该测试会调用 `mark.parametrize, _make_synthetic_sampler, _make_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata`。 代码主体包含 1 个显式断言。

### test_synthetic_all_rejected (lines 978-996)
```python
def test_synthetic_all_rejected(all_greedy: bool):
    """With all rates=0.0, the first token is always rejected."""
    sampler = _make_synthetic_sampler([0.0, 0.0])
    spec_tokens = [[1, 2], [3]]
    output_tokens = [[10, 20, 50], [30, 40]]

    metadata = _make_sampling_metadata(all_greedy)
    logits = create_logits_tensor(output_tokens)
    bonus = torch.tensor([50, 40], device=DEVICE_TYPE)
    spec_decode_metadata = create_spec_decode_metadata(spec_tokens, logits)

    mock_sampler_output(sampler, bonus)
    output = sampler(spec_decode_metadata, None, logits, metadata)
    result = output.sampled_token_ids
    # Exactly one token emitted per sequence (the rejection fallback),
    # followed by placeholders.
    for row in result:
        assert row[0] != PLACEHOLDER_TOKEN_ID
        assert (row[1:] == PLACEHOLDER_TOKEN_ID).all()
```
**EN:** Parameterized test covering `synthetic all rejected`. Parameter axes: `all_greedy`. Inputs/fixtures: `all_greedy`. It exercises `mark.parametrize, _make_synthetic_sampler, _make_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `synthetic all rejected` 的测试用例。 参数维度：`all_greedy`。 输入或 fixture：`all_greedy`。 该测试会调用 `mark.parametrize, _make_synthetic_sampler, _make_sampling_metadata, create_logits_tensor, torch.tensor, create_spec_decode_metadata`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Logits post-processing behavior
- **CN:** logits 后处理行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, torch.nn.functional`.
- **CN:** 外部库：`pytest, torch, torch.nn.functional`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.rejection_sampler, vllm.v1.sample.sampler, vllm.v1.spec_decode.metadata`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.rejection_sampler, vllm.v1.sample.sampler, vllm.v1.spec_decode.metadata`。
- **EN:** Local test helpers: `tests.v1.sample.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.sample.utils`。
- **EN:** Standard-library support: `typing, unittest.mock`.
- **CN:** 标准库支持：`typing, unittest.mock`。
