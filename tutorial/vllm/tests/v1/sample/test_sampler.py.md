# test_sampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/sample/test_sampler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `sampler` behavior and regressions in the v1 stack. / 验证 v1 栈中 `sampler` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-14)
```python
import numpy as np
import pytest
import torch

from tests.v1.sample.utils import create_allowed_token_ids
from vllm.platforms import current_platform
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.utils.torch_utils import make_tensor_with_pad
from vllm.v1.sample.logits_processor import LogitsProcessors
from vllm.v1.sample.metadata import SamplingMetadata
from vllm.v1.sample.sampler import Sampler
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch`. vLLM modules under test include `vllm.platforms, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, ...`. Local helpers come from `tests.v1.sample.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, ...`。 本地测试辅助逻辑来自 `tests.v1.sample.utils`。

### Module state / 模块级状态 (lines 16-25)
```python
PIN_MEMORY_AVAILABLE = is_pin_memory_available()
MAX_NUM_REQS = 256
VOCAB_SIZE = 1024
NUM_OUTPUT_TOKENS = 20
DEVICE_TYPE = current_platform.device_type
DEVICES = [
    f"{DEVICE_TYPE}:{i}"
    for i in range(1 if current_platform.device_count() == 1 else 2)
]
MAX_NUM_PROMPT_TOKENS = 64
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `PIN_MEMORY_AVAILABLE, MAX_NUM_REQS, VOCAB_SIZE, NUM_OUTPUT_TOKENS, DEVICE_TYPE, DEVICES, ...`. Shared setup calls include `is_pin_memory_available, range, current_platform.device_count`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`PIN_MEMORY_AVAILABLE, MAX_NUM_REQS, VOCAB_SIZE, NUM_OUTPUT_TOKENS, DEVICE_TYPE, DEVICES, ...`。 共享初始化调用包括 `is_pin_memory_available, range, current_platform.device_count`。

### _create_fake_logits (lines 28-30)
```python
def _create_fake_logits(batch_size: int, vocab_size: int) -> torch.Tensor:
    fake_logits = torch.full((batch_size, vocab_size), 1e-2, dtype=torch.float)
    return fake_logits
```
**EN:** Helper function `_create_fake_logits` encapsulates reusable logic for `fake logits`. Inputs: `batch_size, vocab_size`. Key calls include `torch.full`.
**CN:** 辅助函数 `_create_fake_logits` 封装了与 `fake logits` 相关的可复用逻辑。 输入参数：`batch_size, vocab_size`。 关键调用包括 `torch.full`。

### _create_penalty_tensor (lines 33-38)
```python
def _create_penalty_tensor(
    batch_size: int, penalty_value: float, device: torch.device
) -> torch.Tensor:
    return torch.full(
        (batch_size,), fill_value=penalty_value, dtype=torch.float, device=device
    )
```
**EN:** Helper function `_create_penalty_tensor` encapsulates reusable logic for `penalty tensor`. Inputs: `batch_size, penalty_value, device`. Key calls include `torch.full`.
**CN:** 辅助函数 `_create_penalty_tensor` 封装了与 `penalty tensor` 相关的可复用逻辑。 输入参数：`batch_size, penalty_value, device`。 关键调用包括 `torch.full`。

### _create_prompt_tokens_tensor (lines 41-52)
```python
def _create_prompt_tokens_tensor(
    prompt_token_ids: list[list[int]],
    vocab_size: int,
    device: torch.device,
) -> torch.Tensor:
    return make_tensor_with_pad(
        prompt_token_ids,
        pad=vocab_size,
        device=device,
        dtype=torch.int64,
        pin_memory=False,
    )
```
**EN:** Helper function `_create_prompt_tokens_tensor` encapsulates reusable logic for `prompt tokens tensor`. Inputs: `prompt_token_ids, vocab_size, device`. Key calls include `make_tensor_with_pad`.
**CN:** 辅助函数 `_create_prompt_tokens_tensor` 封装了与 `prompt tokens tensor` 相关的可复用逻辑。 输入参数：`prompt_token_ids, vocab_size, device`。 关键调用包括 `make_tensor_with_pad`。

### _create_bad_words_token_ids (lines 55-73)
```python
def _create_bad_words_token_ids(
    batch_size: int,
    vocab_size: int,
    bad_words_lengths: tuple[int, ...],
) -> dict[int, list[list[int]]]:
    bad_words_token_ids = {}
    for batch_idx in range(batch_size):
        token_ids_single_batch = []
        for bad_words_length in bad_words_lengths:
            token_ids = np.random.choice(
                vocab_size, size=bad_words_length, replace=True
            ).tolist()
            token_ids_single_batch.append(token_ids)
        bad_words_token_ids[batch_idx] = token_ids_single_batch
    if batch_size >= 2:
        # Test no bad_words for some batch
        no_bad_words_batch_idx = np.random.choice(batch_size)
        bad_words_token_ids.pop(no_bad_words_batch_idx, None)
    return bad_words_token_ids
```
**EN:** Helper function `_create_bad_words_token_ids` encapsulates reusable logic for `bad words token ids`. Inputs: `batch_size, vocab_size, bad_words_lengths`. Key calls include `range, random.choice, bad_words_token_ids.pop, choice.tolist, token_ids_single_batch.append`.
**CN:** 辅助函数 `_create_bad_words_token_ids` 封装了与 `bad words token ids` 相关的可复用逻辑。 输入参数：`batch_size, vocab_size, bad_words_lengths`。 关键调用包括 `range, random.choice, bad_words_token_ids.pop, choice.tolist, token_ids_single_batch.append`。

### _collect_suffixes_with_same_prefix (lines 78-81)
```python
def _collect_suffixes_with_same_prefix(
    given_prefix: list[int], bad_words_token_ids: list[list[int]]
) -> list[int]:
    return [bwt[-1] for bwt in bad_words_token_ids if bwt[:-1] == given_prefix]
```
**EN:** Helper function `_collect_suffixes_with_same_prefix` encapsulates reusable logic for `collect suffixes with same prefix`. Inputs: `given_prefix, bad_words_token_ids`.
**CN:** 辅助函数 `_collect_suffixes_with_same_prefix` 封装了与 `collect suffixes with same prefix` 相关的可复用逻辑。 输入参数：`given_prefix, bad_words_token_ids`。

### _generate_valid_token_id (lines 85-94)
```python
def _generate_valid_token_id(
    bad_words_token_ids: list[list[int]], vocab_size: int
) -> int:
    forbidden_start_tokens = set()
    for bad_word in bad_words_token_ids:
        forbidden_start_tokens.add(bad_word[0])
    # Get a safe token that's not in forbidden starts
    safe_token_candidates = list(set(range(vocab_size)) - forbidden_start_tokens)
    # Pick a random safe token
    return np.random.choice(safe_token_candidates)
```
**EN:** Helper function `_generate_valid_token_id` encapsulates reusable logic for `generate valid token id`. Inputs: `bad_words_token_ids, vocab_size`. Key calls include `set, list, random.choice, forbidden_start_tokens.add, range`.
**CN:** 辅助函数 `_generate_valid_token_id` 封装了与 `generate valid token id` 相关的可复用逻辑。 输入参数：`bad_words_token_ids, vocab_size`。 关键调用包括 `set, list, random.choice, forbidden_start_tokens.add, range`。

### _update_output_token_ids_for_bad_words (lines 97-125)
```python
def _update_output_token_ids_for_bad_words(
    metadata: SamplingMetadata, vocab_size: int
) -> dict[int, list[int]]:
    bad_words_last_tokens = {}
    for batch_idx, bad_words_token_ids in metadata.bad_words_token_ids.items():
        output_token_ids = metadata.output_token_ids[batch_idx]
        bad_words_last_token: list[int] = []
        for i, bad_word_token_ids in enumerate(bad_words_token_ids):
            if len(bad_word_token_ids) == 1:
                # Single token id always affects logits
                bad_words_last_token.append(bad_word_token_ids[0])
            else:
                prefix_length = len(bad_word_token_ids) - 1
                has_bad_words = np.random.choice([True, False])
                if has_bad_words:
                    prefix = bad_word_token_ids[:-1]
                    output_token_ids[-prefix_length:] = prefix
                    # Collect all last tokens from other bad words
                    # that share this prefix
                    bad_words_last_token.extend(
                        _collect_suffixes_with_same_prefix(prefix, bad_words_token_ids)
                    )
                    break  # Maximum one update to output_token_ids
                else:  # Make sure no accidental match to bad words
                    output_token_ids[-1] = _generate_valid_token_id(
                        bad_words_token_ids, vocab_size
                    )
        bad_words_last_tokens[batch_idx] = bad_words_last_token
    return bad_words_last_tokens
```
**EN:** Helper function `_update_output_token_ids_for_bad_words` encapsulates reusable logic for `update output token ids for bad words`. Inputs: `metadata, vocab_size`. Key calls include `bad_words_token_ids.items, enumerate, len, bad_words_last_token.append, random.choice, bad_words_last_token.extend`.
**CN:** 辅助函数 `_update_output_token_ids_for_bad_words` 封装了与 `update output token ids for bad words` 相关的可复用逻辑。 输入参数：`metadata, vocab_size`。 关键调用包括 `bad_words_token_ids.items, enumerate, len, bad_words_last_token.append, random.choice, bad_words_last_token.extend`。

### _create_default_sampling_metadata (lines 128-166)
```python
def _create_default_sampling_metadata(
    num_output_tokens: int,
    batch_size: int,
    vocab_size: int,
    device: torch.device,
) -> SamplingMetadata:
    output_token_ids: list[list[int]] = []
    prompt_token_ids: list[list[int]] = []
    for _ in range(batch_size):
        output_token_ids.append(
            np.random.randint(0, vocab_size, size=num_output_tokens).tolist()
        )
        prompt_token_ids.append(
            np.random.randint(
                0, vocab_size, size=np.random.randint(1, MAX_NUM_PROMPT_TOKENS)
            ).tolist()
    fake_sampling_metadata = SamplingMetadata(
    # ... excerpt omitted for brevity ...
        no_penalties=True,
        allowed_token_ids_mask=None,
        bad_words_token_ids={},
        logitsprocs=LogitsProcessors(),
    )
    return fake_sampling_metadata
```
**EN:** Helper function `_create_default_sampling_metadata` encapsulates reusable logic for `default sampling metadata`. Inputs: `num_output_tokens, batch_size, vocab_size, device`. Key calls include `range, SamplingMetadata, output_token_ids.append, prompt_token_ids.append, randint.tolist, torch.full`.
**CN:** 辅助函数 `_create_default_sampling_metadata` 封装了与 `default 采样 metadata` 相关的可复用逻辑。 输入参数：`num_output_tokens, batch_size, vocab_size, device`。 关键调用包括 `range, SamplingMetadata, output_token_ids.append, prompt_token_ids.append, randint.tolist, torch.full`。

### _create_weighted_output_token_list (lines 169-200)
```python
def _create_weighted_output_token_list(
    batch_size: int, vocab_size: int
) -> tuple[list[list[int]], list[list[int]]]:
    """
    Creates an output token list where each token occurs a distinct
    number of times.

    For each batch, a random subset of token IDs is selected from the
    vocabulary. The selected tokens are then added to the output token
    list, each with a different frequency.

    Returns:
        tuple[list[list[int]], list[list[int]]]:
            - The first element is the output token list, where each sublist
              corresponds to a batch and contains tokens with weighted
              frequencies.
            - The second element is a list of distinct token IDs for each
              batch, ordered by their frequency in the corresponding output
              list.
    """
    output_token_ids: list[list[int]] = []
    sorted_token_ids_in_output: list[list[int]] = []
    for _ in range(batch_size):
        distinct_token_ids = np.random.choice(
            vocab_size, size=np.random.randint(1, 10), replace=False
        ).tolist()
        sorted_token_ids_in_output.append(distinct_token_ids)
        output_token_ids_for_batch = []
        for index, token_id in enumerate(distinct_token_ids):
            output_token_ids_for_batch.extend([token_id for _ in range(index + 1)])
        output_token_ids.append(output_token_ids_for_batch)
    return output_token_ids, sorted_token_ids_in_output
```
**EN:** Helper function `_create_weighted_output_token_list` encapsulates reusable logic for `weighted output token list`. Inputs: `batch_size, vocab_size`. Key calls include `range, choice.tolist, sorted_token_ids_in_output.append, enumerate, output_token_ids.append, output_token_ids_for_batch.extend`.
**CN:** 辅助函数 `_create_weighted_output_token_list` 封装了与 `weighted output token list` 相关的可复用逻辑。 输入参数：`batch_size, vocab_size`。 关键调用包括 `range, choice.tolist, sorted_token_ids_in_output.append, enumerate, output_token_ids.append, output_token_ids_for_batch.extend`。

### test_sampler_presence_penalty (lines 206-250)
```python
def test_sampler_presence_penalty(
    device: str, batch_size: int, presence_penalty: float
):
    """
    Test to verify that if presence penalty is enabled then tokens
    are penalized as per their presence in the existing output.
    torch.set_default_device(device)
    # Create fake logits where each token is assigned the same
    # logit value.
    fake_logits = _create_fake_logits(batch_size, VOCAB_SIZE)
    sampling_metadata = _create_default_sampling_metadata(
        NUM_OUTPUT_TOKENS, batch_size, VOCAB_SIZE, torch.device(device)
    )
    output_token_ids = sampling_metadata.output_token_ids
    sampling_metadata.presence_penalties = _create_penalty_tensor(
        batch_size, presence_penalty, torch.device(device)
    # ... excerpt omitted for brevity ...
            assert penalized_token_id in output_token_ids[batch_idx]
            assert non_penalized_token_id not in output_token_ids[batch_idx]
            # If `presence_penalty` is set to a value less than 0, it indicates
            # a preference for existing tokens over new ones. Verify that the
            # non-penalized token ID exists in the output, while the penalized
            # token ID does not.
            assert non_penalized_token_id in output_token_ids[batch_idx]
            assert penalized_token_id not in output_token_ids[batch_idx]
```
**EN:** Parameterized test covering `sampler presence penalty`. Parameter axes: `device, batch_size, presence_penalty`. Inputs/fixtures: `device, batch_size, presence_penalty`. It exercises `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_penalty_tensor, Sampler`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `sampler presence penalty` 的测试用例。 参数维度：`device, batch_size, presence_penalty`。 输入或 fixture：`device, batch_size, presence_penalty`。 该测试会调用 `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_penalty_tensor, Sampler`。 代码主体包含 4 个显式断言。

### test_sampler_frequency_penalty (lines 256-306)
```python
def test_sampler_frequency_penalty(
    device: str, batch_size: int, frequency_penalty: float
):
    """
    Test to verify that if frequency penalty is enabled then tokens are
    penalized as per their frequency of occurrence.
    torch.set_default_device(device)
    # Create fake logits where each token is assigned the same
    # logit value.
    fake_logits = _create_fake_logits(batch_size, VOCAB_SIZE)
    sampling_metadata = _create_default_sampling_metadata(
        NUM_OUTPUT_TOKENS, batch_size, VOCAB_SIZE, torch.device(device)
    )
    sampling_metadata.frequency_penalties = _create_penalty_tensor(
        batch_size, frequency_penalty, torch.device(device)
    output_token_ids, sorted_token_ids_in_output = _create_weighted_output_token_list(
    # ... excerpt omitted for brevity ...
            assert non_penalized_token_id not in distinct_sorted_token_ids_in_output
            assert penalized_token_id == most_frequent_token_id
            # a preference for existing tokens over new ones. Verify that the
            # non-penalized token ID is the one that occurs most frequently
            # in the output, while the penalized token ID is one that has not
            # yet appeared.
            assert non_penalized_token_id == most_frequent_token_id
            assert penalized_token_id not in distinct_sorted_token_ids_in_output
```
**EN:** Parameterized test covering `sampler frequency penalty`. Parameter axes: `device, batch_size, frequency_penalty`. Inputs/fixtures: `device, batch_size, frequency_penalty`. It exercises `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_penalty_tensor, _create_weighted_output_token_list`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `sampler frequency penalty` 的测试用例。 参数维度：`device, batch_size, frequency_penalty`。 输入或 fixture：`device, batch_size, frequency_penalty`。 该测试会调用 `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_penalty_tensor, _create_weighted_output_token_list`。 代码主体包含 4 个显式断言。

### test_sampler_repetition_penalty (lines 312-364)
```python
def test_sampler_repetition_penalty(
    device: str, batch_size: int, repetition_penalty: float
):
    """
    Test to verify that when the repetition penalty is enabled, tokens
    are penalized based on their presence in the prompt or the existing
    output.
    torch.set_default_device(device)
    # Create fake logits where each token is assigned the same
    # logit value.
    fake_logits = _create_fake_logits(batch_size, VOCAB_SIZE)
    sampling_metadata = _create_default_sampling_metadata(
        NUM_OUTPUT_TOKENS, batch_size, VOCAB_SIZE, torch.device(device)
    )
    sampling_metadata.repetition_penalties = _create_penalty_tensor(
        batch_size, repetition_penalty, torch.device(device)
    # ... excerpt omitted for brevity ...
            assert (
                and penalized_token_id not in output_tokens
            )
                non_penalized_token_id in prompt_tokens
                or non_penalized_token_id in output_tokens
```
**EN:** Parameterized test covering `sampler repetition penalty`. Parameter axes: `device, batch_size, repetition_penalty`. Inputs/fixtures: `device, batch_size, repetition_penalty`. It exercises `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_penalty_tensor, Sampler`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `sampler repetition penalty` 的测试用例。 参数维度：`device, batch_size, repetition_penalty`。 输入或 fixture：`device, batch_size, repetition_penalty`。 该测试会调用 `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_penalty_tensor, Sampler`。 代码主体包含 4 个显式断言。

### test_sampler_allowed_token_ids (lines 370-410)
```python
def test_sampler_allowed_token_ids(
    device: str, batch_size: int, num_allowed_token_ids: int
):
    """
    Test to verify that when the repetition penalty is enabled, tokens
    are penalized based on their presence in the prompt or the existing
    output.
    torch.set_default_device(device)
    # Create fake logits where each token is assigned the same
    # logit value.
    fake_logits = _create_fake_logits(batch_size, VOCAB_SIZE)
    sampling_metadata = _create_default_sampling_metadata(
        NUM_OUTPUT_TOKENS, batch_size, VOCAB_SIZE, torch.device(device)
    )
    mask = create_allowed_token_ids(
        batch_size=batch_size,
        vocab_size=VOCAB_SIZE,
    # ... excerpt omitted for brevity ...
            assert torch.all(logits_for_req != -float("inf"))
            if token_id >= start and token_id < end:
                assert logits_for_req[token_id] == -float("inf"), (
                    f"{batch_idx}, {token_id}"
                )
            else:
                assert logits_for_req[token_id] != -float("inf")
```
**EN:** Parameterized test covering `sampler allowed token ids`. Parameter axes: `device, batch_size, num_allowed_token_ids`. Inputs/fixtures: `device, batch_size, num_allowed_token_ids`. It exercises `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, create_allowed_token_ids, Sampler`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `sampler allowed token ids` 的测试用例。 参数维度：`device, batch_size, num_allowed_token_ids`。 输入或 fixture：`device, batch_size, num_allowed_token_ids`。 该测试会调用 `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, create_allowed_token_ids, Sampler`。 代码主体包含 3 个显式断言。

### test_sampler_bad_words (lines 416-450)
```python
def test_sampler_bad_words(
    device: str, batch_size: int, bad_words_lengths: tuple[int, ...]
):
    """
    Test to verify that when the bad words restriction is present, tokens
    are penalized based on their match with the bad words.
    torch.set_default_device(device)
    # Create fake logits where each token is assigned the same
    # logit value.
    fake_logits = _create_fake_logits(batch_size, VOCAB_SIZE)
    sampling_metadata = _create_default_sampling_metadata(
        NUM_OUTPUT_TOKENS, batch_size, VOCAB_SIZE, torch.device(device)
    )
    sampling_metadata.bad_words_token_ids = _create_bad_words_token_ids(
        batch_size, VOCAB_SIZE, bad_words_lengths
    bad_words_last_tokens = _update_output_token_ids_for_bad_words(
    # ... excerpt omitted for brevity ...
                batch_idx in bad_words_last_tokens
                and token_id in bad_words_last_tokens[batch_idx]
            ):
                assert logits_for_req[token_id] == -float("inf")
            else:
                assert logits_for_req[token_id] != -float("inf")
```
**EN:** Parameterized test covering `sampler bad words`. Parameter axes: `device, batch_size, bad_words_lengths`. Inputs/fixtures: `device, batch_size, bad_words_lengths`. It exercises `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_bad_words_token_ids, _update_output_token_ids_for_bad_words`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `sampler bad words` 的测试用例。 参数维度：`device, batch_size, bad_words_lengths`。 输入或 fixture：`device, batch_size, bad_words_lengths`。 该测试会调用 `mark.parametrize, torch.set_default_device, _create_fake_logits, _create_default_sampling_metadata, _create_bad_words_token_ids, _update_output_token_ids_for_bad_words`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Logits post-processing behavior
- **CN:** logits 后处理行为
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch`.
- **CN:** 外部库：`numpy, pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.sampler`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm.utils.platform_utils, vllm.utils.torch_utils, vllm.v1.sample.logits_processor, vllm.v1.sample.metadata, vllm.v1.sample.sampler`。
- **EN:** Local test helpers: `tests.v1.sample.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.sample.utils`。
