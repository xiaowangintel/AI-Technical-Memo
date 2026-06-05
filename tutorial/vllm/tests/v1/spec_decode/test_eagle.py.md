# test_eagle.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/spec_decode/test_eagle.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `eagle` behavior and regressions in the v1 stack. / 验证 v1 栈中 `eagle` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-35)
```python
from unittest import mock

import numpy as np
import pytest
import torch

from tests.utils import get_attn_backend_list_based_on_platform
from tests.v1.attention.utils import (
    BatchSpec,
    create_common_attn_metadata,
    create_standard_kv_cache_spec,
    try_get_attention_backend,
)
from vllm.config import (
    AttentionConfig,
    CacheConfig,
    DeviceConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    SpeculativeConfig,
    VllmConfig,
)
from vllm.config.load import LoadConfig
from vllm.model_executor.models.llama import LlamaForCausalLM
from vllm.platforms import current_platform
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.spec_decode.dflash import DFlashProposer
from vllm.v1.spec_decode.draft_model import DraftModelProposer
from vllm.v1.spec_decode.eagle import EagleProposer
from vllm.v1.spec_decode.metadata import SpecDecodeMetadata
from vllm.v1.worker.gpu_input_batch import CachedRequestState, InputBatch
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest, torch`. vLLM modules under test include `vllm.config, vllm.config.load, vllm.model_executor.models.llama, vllm.platforms, vllm.v1.attention.backends.registry, ...`. Local helpers come from `tests.utils, tests.v1.attention.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest, torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.config.load, vllm.model_executor.models.llama, vllm.platforms, vllm.v1.attention.backends.registry, ...`。 本地测试辅助逻辑来自 `tests.utils, tests.v1.attention.utils`。

### Module state / 模块级状态 (lines 37-45)
```python
model_dir = "meta-llama/Llama-3.1-8B-Instruct"
eagle_dir = "yuhuili/EAGLE-LLaMA3.1-Instruct-8B"
eagle3_dir = "yuhuili/EAGLE3-LLaMA3.1-Instruct-8B"
ar_draft_model_dir = "amd/PARD-Llama-3.2-1B"  # Compatible with parallel and AR drafting
dflash_target_dir = "Qwen/Qwen3-8B"
dflash_dir = "z-lab/Qwen3-8B-DFlash-b16"

BLOCK_SIZE = 16
DEVICE_TYPE = current_platform.device_type
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `model_dir, eagle_dir, eagle3_dir, ar_draft_model_dir, dflash_target_dir, dflash_dir, ...`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`model_dir, eagle_dir, eagle3_dir, ar_draft_model_dir, dflash_target_dir, dflash_dir, ...`。

### _create_proposer (lines 48-115)
```python
def _create_proposer(
    method: str,
    num_speculative_tokens: int,
    attention_backend: str | None = None,
    parallel_drafting: bool = False,
    rejection_sample_method: str = "standard",
    draft_sample_method: str = "greedy",
) -> EagleProposer:
    # Method-dependent setup
    if method == "eagle":
        target_model_dir = model_dir
        draft_model_dir = eagle_dir
    elif method == "eagle3":
        draft_model_dir = eagle3_dir
    elif method == "draft_model":
        draft_model_dir = ar_draft_model_dir
    # ... excerpt omitted for brevity ...
    elif "eagle" in method:
        proposer = EagleProposer(vllm_config=vllm_config, device=device)
    else:
        proposer = DraftModelProposer(vllm_config=vllm_config, device=device)
    proposer.block_size = BLOCK_SIZE
    return proposer
```
**EN:** Helper function `_create_proposer` encapsulates reusable logic for `proposer`. Inputs: `method, num_speculative_tokens, attention_backend, parallel_drafting, rejection_sample_method, draft_sample_method`. Key calls include `ModelConfig, SpeculativeConfig, VllmConfig, DFlashProposer, ParallelConfig, CacheConfig`.
**CN:** 辅助函数 `_create_proposer` 封装了与 `proposer` 相关的可复用逻辑。 输入参数：`method, num_speculative_tokens, attention_backend, parallel_drafting, rejection_sample_method, draft_sample_method`。 关键调用包括 `ModelConfig, SpeculativeConfig, VllmConfig, DFlashProposer, ParallelConfig, CacheConfig`。

### test_prepare_next_token_ids (lines 118-195)
```python
def test_prepare_next_token_ids():
    """
    Test for prepare_next_token_ids_cpu and prepare_next_token_ids_padded.
    Each will produce a device tensor of next_token_ids, taking as input
    either the GPU tensor of sampled_token_ids with -1 for rejected tokens,
    or the CPU python list[list[int]] with the rejected tokens removed.
    device = torch.device(DEVICE_TYPE)

    num_requests = 4
    num_speculative_tokens = 4
    req_ids = [f"req_{i + 1}" for i in range(num_requests)]
    mock_input_batch = mock.MagicMock(spec=InputBatch)
    mock_input_batch.req_ids = req_ids
    mock_input_batch.num_reqs = num_requests
    mock_input_batch.vocab_size = 100
    mock_input_batch.num_tokens_no_spec = np.array(
        [num_speculative_tokens + 1] * num_requests
    # ... excerpt omitted for brevity ...
        mock_request = mock.MagicMock(spec=CachedRequestState)
    expected_next_token_ids_cpu = [1, 4, 30, 40]
    expected_next_token_ids_tensor = torch.tensor(
        expected_next_token_ids_cpu, dtype=torch.int32, device=device
    assert torch.equal(next_token_ids_from_cpu, expected_next_token_ids_tensor)
    expected_valid_sampled_tokens_count = torch.tensor(
            discarded_req_mask,
        )
    )
    assert torch.equal(next_token_ids_from_padded, expected_next_token_ids_tensor)
    assert torch.equal(valid_sampled_tokens_count, expected_valid_sampled_tokens_count)
```
**EN:** Test case covering `prepare next token ids`. It exercises `torch.device, mock.MagicMock, np.array, torch.tensor, range, _create_proposer`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `prepare next token ids` 的测试用例。 该测试会调用 `torch.device, mock.MagicMock, np.array, torch.tensor, range, _create_proposer`。 代码主体包含 3 个显式断言。

### test_prepare_inputs (lines 198-285)
```python
def test_prepare_inputs():
    """
    cu_target_query_lens: [0, a, a + b, a + b + c]
    num_rejected_tokens: [n1, n2, n3]
    num_tokens_per_req: [a - n1, b - n2, c - n3]
    cu_num_tokens: [0, a - n1, a + b - n1 - n2, a + b + c - n1 - n2 - n3]
    token_indices: [0, 1, ..., a - n1 - 1,
                    a, a + 1, ..., a + b - n2 - 1,
                    a + b, a + b + 1, ..., a + b + c - n3 - 1]
    device = torch.device(DEVICE_TYPE)

    # q1 = 4, q2 = 7, q3 = 5
    # n1 = 1, n2 = 3, n3 = 2
    batch_spec = BatchSpec(
        seq_lens=[4, 7, 5],
        query_lens=[4, 7, 5],
    # ... excerpt omitted for brevity ...
    expected_cu_num_tokens = torch.tensor(
    expected_token_indices = torch.tensor(
        common_attn_metadata, sampled_token_ids, num_draft_tokens
    )
    assert torch.equal(updated_metadata.query_start_loc, expected_cu_num_tokens)
    assert token_indices.shape[0] == expected_cu_num_tokens[-1].item()
    assert torch.equal(token_indices, expected_token_indices)
```
**EN:** Test case covering `prepare inputs`. It exercises `torch.device, BatchSpec, create_common_attn_metadata, torch.tensor, _create_proposer, proposer.prepare_inputs`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `prepare inputs` 的测试用例。 该测试会调用 `torch.device, BatchSpec, create_common_attn_metadata, torch.tensor, _create_proposer, proposer.prepare_inputs`。 代码主体包含 3 个显式断言。

### test_prepare_inputs_padded (lines 288-349)
```python
def test_prepare_inputs_padded():
    """
    Input scenario is 3 requests with num_speculative_tokens == 2 and:
    - Request 1: query_len = 3, rejected = 1
    - Request 2: query_len = 3, rejected = 0
    - Request 3: query_len = 3, rejected = 2

    Expected outputs:
    token_indices_to_sample: [1, 5, 6]
    Reason: After accounting for rejections, these are the valid token positions
            from the original indices to sample from.
    device = torch.device(DEVICE_TYPE)
    expected_token_indices_to_sample = torch.tensor(
        [1, 5, 6], dtype=torch.int32, device=device
    )
    # ... excerpt omitted for brevity ...
    # Needed for cu_num_draft_tokens, which is expected to be [3, 6, 9]
    expected_query_start_loc = torch.tensor(
    expected_num_rejected = torch.tensor([1, 0, 2], dtype=torch.int32, device=device)
    assert torch.equal(num_rejected_tokens_gpu, expected_num_rejected)
    assert output_metadata.max_query_len == 3
    assert torch.equal(output_metadata.query_start_loc, expected_query_start_loc)
    assert torch.equal(token_indices_to_sample, expected_token_indices_to_sample)
```
**EN:** Test case covering `prepare inputs padded`. It exercises `torch.device, torch.tensor, BatchSpec, create_common_attn_metadata, SpecDecodeMetadata.make_dummy, _create_proposer`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prepare inputs padded` 的测试用例。 该测试会调用 `torch.device, torch.tensor, BatchSpec, create_common_attn_metadata, SpecDecodeMetadata.make_dummy, _create_proposer`。 代码主体包含 4 个显式断言。

### test_set_inputs_first_pass_default_eagle (lines 352-435)
```python
def test_set_inputs_first_pass_default_eagle():
    """
    Test for set_inputs_first_pass without extra input slots (default EAGLE).

    This tests the path where needs_extra_input_slots=False, which is the
    default EAGLE pathway. In this case:
    - Input IDs are rotated (shifted by one)
    - The next_token_ids are inserted at the last position of each request
    - Positions are copied as-is
    - Hidden states are copied as-is
    - The CommonAttentionMetadata is returned unchanged
    Setup:
    - 3 requests with query_lens [3, 2, 4]
    - Tokens: [a1, a2, a3, b1, b2, c1, c2, c3, c4]
    - After rotation: [a2, a3, -, b2, -, c2, c3, c4, -]
    - After inserting next_tokens [100, 200, 300]:
        [a2, a3, 100, b2, 200, c2, c3, c4, 300]
    # ... excerpt omitted for brevity ...
    assert num_tokens == 9  # Total tokens unchanged
    expected_token_indices_to_sample = torch.tensor(
    assert torch.equal(token_indices_to_sample, expected_token_indices_to_sample)
    assert output_cad is common_attn_metadata
    expected_input_ids = torch.tensor(
    assert torch.equal(proposer.input_ids[:num_tokens], expected_input_ids)
    # Verify positions are copied as-is
    assert torch.equal(proposer.positions[:num_tokens], target_positions)
    # Verify hidden states are copied as-is
    assert torch.equal(proposer.hidden_states[:num_tokens], target_hidden_states)
```
**EN:** Test case covering `set inputs first pass default eagle`. It exercises `torch.device, _create_proposer, BatchSpec, create_common_attn_metadata, torch.tensor, torch.randn`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `set inputs first pass default eagle` 的测试用例。 该测试会调用 `torch.device, _create_proposer, BatchSpec, create_common_attn_metadata, torch.tensor, torch.randn`。 代码主体包含 6 个显式断言。

### test_set_inputs_first_pass_draft_model (lines 438-577)
```python
def test_set_inputs_first_pass_draft_model():
    """
    Test for set_inputs_first_pass with a draft model (extra input slots,
    no shift).

    This tests the path where needs_extra_input_slots=True and
    shift_input_ids=False (draft model case). In this case:
    - Input IDs are NOT shifted
    - Each request gets extra_slots_per_request (1) new slots
    - The kernel handles copying tokens and inserting bonus/padding tokens
    - A new CommonAttentionMetadata is returned with updated query_start_loc
    Setup:
    - 2 requests
    - Request 0: tokens [10, 11, 12] at positions [0, 1, 2]
      - Only tokens [10, 11] are "valid" (query_end_loc=1),
        token 12 is a rejected token from previous speculation
    - Request 1: tokens [20, 21] at positions [0, 1], both valid.
    # ... excerpt omitted for brevity ...
    mock_kv_cache_spec = mock.MagicMock()
    mock_attn_group = mock.MagicMock()
    assert proposer.net_num_new_slots_per_request == 1
    assert proposer.needs_extra_input_slots
    assert num_tokens == 7
    expected_input_ids = torch.tensor(
    assert torch.equal(token_indices_to_sample, expected_token_indices_to_sample)
    # Verify the new CAD has updated query_start_loc
    # Original: [0, 3, 5] -> New: [0, 4, 7] (each request gains 1 slot)
    expected_query_start_loc = torch.tensor([0, 4, 7], dtype=torch.int32, device=device)
    assert torch.equal(output_cad.query_start_loc, expected_query_start_loc)
```
**EN:** Test case covering `set inputs first pass draft model`. It exercises `torch.device, _create_proposer, torch.zeros, mock.MagicMock, BatchSpec, create_common_attn_metadata`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `set inputs first pass draft model` 的测试用例。 该测试会调用 `torch.device, _create_proposer, torch.zeros, mock.MagicMock, BatchSpec, create_common_attn_metadata`。 代码主体包含 9 个显式断言。

### test_set_inputs_first_pass_parallel_drafting (lines 580-733)
```python
def test_set_inputs_first_pass_parallel_drafting():
    """
    Test for set_inputs_first_pass with parallel drafting (extra input slots,
    with shift).

    This tests the path where needs_extra_input_slots=True and
    shift_input_ids=True (parallel drafting case). In this case:
    - Input IDs ARE shifted (like default EAGLE)
    - Each request gets extra_slots_per_request (3) new slots
    - Parallel drafting tokens are inserted and marked as masked
    - Hidden states are mapped correctly
    Setup:
    - 2 requests with query_lens [4, 4] (1 bonus + 3 spec tokens each)
    - Request 0: tokens [10, 11, 12, 13] at positions [5, 6, 7, 8]
      - Only tokens [10, 11, 12] are "valid", token 13 is rejected
    - Request 1: tokens [20, 21, 22, 23] at positions [10, 11, 12, 13], all valid.
    - next_token_ids: [100, 200] (bonus tokens)
    # ... excerpt omitted for brevity ...
    mock_kv_cache_spec = mock.MagicMock()
    mock_attn_group = mock.MagicMock()
    assert num_tokens == 12
    expected_input_ids = torch.tensor(
    assert torch.equal(proposer.input_ids[:num_tokens], expected_input_ids)
    expected_positions = torch.tensor(
    parallel_drafting_hs = proposer.parallel_drafting_hidden_state_tensor
    for i in range(num_tokens):
        if expected_is_masked[i]:
            assert torch.equal(proposer.hidden_states[i], parallel_drafting_hs), (
                f"Masked position {i} should have parallel drafting hidden state"
            )
```
**EN:** Test case covering `set inputs first pass parallel drafting`. It exercises `torch.device, _create_proposer, torch.zeros, mock.MagicMock, BatchSpec, create_common_attn_metadata`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `set inputs first pass parallel drafting` 的测试用例。 该测试会调用 `torch.device, _create_proposer, torch.zeros, mock.MagicMock, BatchSpec, create_common_attn_metadata`。 代码主体包含 8 个显式断言。

### test_load_model (lines 744-831)
```python
def test_load_model(
    mock_get_model,
    mock_get_layers,
    mock_get_pp_group,
    method,
    attn_backend,
    pp_size,
    use_distinct_embed_tokens,
    use_distinct_lm_head,
    monkeypatch,
):
    if attn_backend == "ROCM_AITER_FA" and current_platform.is_rocm():
        monkeypatch.setenv("VLLM_ROCM_USE_AITER", "1")

    # Setup draft model mock
    mock_model = mock.MagicMock()
    mock_model.model = mock.MagicMock()
    mock_model.has_own_embed_tokens = use_distinct_embed_tokens
    # ... excerpt omitted for brevity ...
        mock_model.model.embed_tokens = mock.MagicMock()
        mock_model.lm_head = mock.MagicMock()
        "target_attn_1": mock.MagicMock(),
        "target_attn_2": mock.MagicMock(),
    all_attn_layers = {**target_attn_layers, "draft_extra_attn": mock.MagicMock()}
    mock_pp_group = mock.MagicMock()
    # Verify that the embed tokens are set correctly
    # If pp_size is > 1, the embed tokens should be distinct
    if pp_size > 1 or use_distinct_embed_tokens:
        assert proposer.model.model.embed_tokens is not target_model.model.embed_tokens
    else:
        assert proposer.model.model.embed_tokens is target_model.model.embed_tokens
```
**EN:** Parameterized test covering `load model`. Parameter axes: `method, attn_backend, pp_size, use_distinct_embed_tokens, use_distinct_lm_head`. Inputs/fixtures: `mock_get_model, mock_get_layers, mock_get_pp_group, method, attn_backend, pp_size, use_distinct_embed_tokens, use_distinct_lm_head, ...`. It exercises `mark.parametrize, mock.patch, mock.MagicMock, mock.create_autospec, _create_proposer, proposer.load_model`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `load model` 的测试用例。 参数维度：`method, attn_backend, pp_size, use_distinct_embed_tokens, use_distinct_lm_head`。 输入或 fixture：`mock_get_model, mock_get_layers, mock_get_pp_group, method, attn_backend, pp_size, use_distinct_embed_tokens, use_distinct_lm_head, ...`。 该测试会调用 `mark.parametrize, mock.patch, mock.MagicMock, mock.create_autospec, _create_proposer, proposer.load_model`。 代码主体包含 5 个显式断言。

### test_propose (lines 837-1001)
```python
def test_propose(method, attn_backend, num_speculative_tokens, monkeypatch):
    if attn_backend == "TRITON_ATTN" and not current_platform.is_rocm():
        pytest.skip(
            "TRITON_ATTN does not support "
            "multi-token eagle spec decode on current platform"
        )

    if attn_backend == "ROCM_AITER_FA" and current_platform.is_rocm():
        monkeypatch.setenv("VLLM_ROCM_USE_AITER", "1")
    # Use GPU device
    device = torch.device(DEVICE_TYPE)
    # Setup test parameters
    batch_size = 2
    seq_len_1 = 5
    seq_len_2 = 3
    total_tokens = seq_len_1 + seq_len_2
    # ... excerpt omitted for brevity ...
        return logits
    model_mock = mock.MagicMock()
    sampling_metadata = mock.MagicMock()
    proposer.runner = mock.MagicMock()
    mock_attn_group = mock.MagicMock()
    assert result.shape == (batch_size, num_speculative_tokens)
        for i in range(batch_size):
            for j in range(num_speculative_tokens):
                expected_tokens[i, j] = base_token_ids[i] + j
    # Verify all tokens match our expectations
    assert torch.equal(result, expected_tokens)
```
**EN:** Parameterized test covering `propose`. Parameter axes: `method, attn_backend, num_speculative_tokens`. Inputs/fixtures: `method, attn_backend, num_speculative_tokens, monkeypatch`. It exercises `mark.parametrize, torch.device, _create_proposer, mock.MagicMock, range, BatchSpec`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `propose` 的测试用例。 参数维度：`method, attn_backend, num_speculative_tokens`。 输入或 fixture：`method, attn_backend, num_speculative_tokens, monkeypatch`。 该测试会调用 `mark.parametrize, torch.device, _create_proposer, mock.MagicMock, range, BatchSpec`。 代码主体包含 2 个显式断言。

### test_propose_stores_probabilistic_draft_probs (lines 1004-1098)
```python
def test_propose_stores_probabilistic_draft_probs(monkeypatch):
    device = torch.device(DEVICE_TYPE)
    batch_size = 2
    seq_lens = [5, 3]
    total_tokens = sum(seq_lens)
    num_speculative_tokens = 3
    vocab_size = 8

    proposer = _create_proposer(
        "draft_model",
        num_speculative_tokens,
        rejection_sample_method="standard",
        draft_sample_method="probabilistic",
    )
    hidden_size = proposer.hidden_size
    expanded_total_tokens = total_tokens + batch_size
    model_mock = mock.MagicMock()
    # ... excerpt omitted for brevity ...
        return probs.argmax(dim=-1), probs
    proposer.runner = mock.MagicMock()
    mock_attn_group = mock.MagicMock()
    sampling_metadata = mock.MagicMock()
    assert result.shape == (batch_size, num_speculative_tokens)
    assert draft_probs is not None
    assert draft_probs.shape == (batch_size, num_speculative_tokens, vocab_size)
    for step, expected_logits in enumerate(logits_returns):
        assert torch.allclose(
            draft_probs[:, step, :],
            torch.softmax(expected_logits, dim=-1),
        )
```
**EN:** Test case covering `propose stores probabilistic draft probs`. Inputs/fixtures: `monkeypatch`. It exercises `torch.device, sum, _create_proposer, mock.MagicMock, range, monkeypatch.setattr`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `propose stores probabilistic draft probs` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `torch.device, sum, _create_proposer, mock.MagicMock, range, monkeypatch.setattr`。 代码主体包含 4 个显式断言。

### test_set_inputs_first_pass_dflash (lines 1101-1231)
```python
def test_set_inputs_first_pass_dflash():
    """
    Test for DFlash set_inputs_first_pass.

    DFlash uses cross-attention: context tokens become K/V and only
    query tokens (bonus + mask) are Q. This tests the DFlash-specific
    input preparation where:
    - Context hidden states are stored by reference (no copy)
    - Query input_ids are [next_token, mask, mask, ...] per request
    - Context and query positions are written to separate buffers
    - token_indices_to_sample points to mask token positions only
    - A new CommonAttentionMetadata is returned with causal=False
    Setup:
    - 3 requests with query_lens [3, 2, 4]
    - num_speculative_tokens = 3
    - num_query_per_req = 4 (1 bonus + 3 mask tokens)
    - next_token_ids: [100, 200, 300]
    # ... excerpt omitted for brevity ...
    assert num_tokens == 3 * num_query_per_req  # 12
    expected_input_ids = torch.tensor(
    assert torch.equal(proposer.input_ids[:num_tokens], expected_input_ids)
    assert torch.equal(
    expected_query_positions = torch.tensor(
        [0, 4, 8, 12], dtype=torch.int32, device=device
    )
    assert torch.equal(output_cad.query_start_loc, expected_query_start_loc)
    # Verify hidden states (stored by reference, not copied)
    assert proposer._dflash_hidden_states is target_hidden_states
```
**EN:** Test case covering `set inputs first pass dflash`. It exercises `torch.device, _create_proposer, BatchSpec, create_common_attn_metadata, torch.tensor, torch.randn`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `set inputs first pass dflash` 的测试用例。 该测试会调用 `torch.device, _create_proposer, BatchSpec, create_common_attn_metadata, torch.tensor, torch.randn`。 代码主体包含 10 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest, torch`.
- **CN:** 外部库：`numpy, pytest, torch`。
- **EN:** vLLM modules under test: `vllm.config, vllm.config.load, vllm.model_executor.models.llama, vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.spec_decode.dflash, vllm.v1.spec_decode.draft_model, vllm.v1.spec_decode.eagle, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.config.load, vllm.model_executor.models.llama, vllm.platforms, vllm.v1.attention.backends.registry, vllm.v1.spec_decode.dflash, vllm.v1.spec_decode.draft_model, vllm.v1.spec_decode.eagle, ...`。
- **EN:** Local test helpers: `tests.utils, tests.v1.attention.utils`.
- **CN:** 本地测试辅助模块：`tests.utils, tests.v1.attention.utils`。
- **EN:** Standard-library support: `unittest`.
- **CN:** 标准库支持：`unittest`。
