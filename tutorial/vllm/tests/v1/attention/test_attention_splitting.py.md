# test_attention_splitting.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/test_attention_splitting.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `attention splitting` behavior and regressions in the v1 stack. / 验证 v1 栈中 `注意力 splitting` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-18)
```python
import pytest
import torch

from tests.v1.attention.test_attention_backends import BATCH_SPECS
from tests.v1.attention.utils import BatchSpec, create_common_attn_metadata
from vllm.v1.attention.backends.utils import (
    split_decodes_and_prefills,
)
from vllm.v1.worker.ubatch_utils import (
    UBatchSlice,
    _make_metadata_with_slice,
    maybe_create_ubatch_slices,
    slice_query_start_locs,
    split_attn_metadata,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, numpy`. vLLM modules under test include `vllm.v1.attention.backends.utils, vllm.v1.worker.ubatch_utils`. Local helpers come from `tests.v1.attention.test_attention_backends, tests.v1.attention.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, numpy`。 被测试的 vLLM 模块包括 `vllm.v1.attention.backends.utils, vllm.v1.worker.ubatch_utils`。 本地测试辅助逻辑来自 `tests.v1.attention.test_attention_backends, tests.v1.attention.utils`。

### sample_query_start_loc (lines 22-24)
```python
def sample_query_start_loc():
    """Sample query_start_loc tensor for testing"""
    return torch.tensor([0, 5, 12, 20, 35, 50])
```
**EN:** Fixture/helper `sample_query_start_loc` prepares reusable state for downstream tests. Key calls include `torch.tensor`.
**CN:** `sample_query_start_loc` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `torch.tensor`。

### test_basic_slice_middle (lines 27-33)
```python
def test_basic_slice_middle(sample_query_start_loc):
    """Test slicing from middle of tensor"""
    req_slice = slice(1, 3)  # slice from index 1 to 3
    result = slice_query_start_locs(sample_query_start_loc, req_slice)

    expected = torch.tensor([0, 7, 15])
    assert torch.equal(result, expected)
```
**EN:** Test case covering `basic slice middle`. Inputs/fixtures: `sample_query_start_loc`. It exercises `slice, slice_query_start_locs, torch.tensor, torch.equal`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `basic slice middle` 的测试用例。 输入或 fixture：`sample_query_start_loc`。 该测试会调用 `slice, slice_query_start_locs, torch.tensor, torch.equal`。 代码主体包含 1 个显式断言。

### test_slice_from_beginning (lines 36-42)
```python
def test_slice_from_beginning(sample_query_start_loc):
    """Test slicing from the beginning of tensor"""
    req_slice = slice(0, 2)  # slice from index 0 to 2
    result = slice_query_start_locs(sample_query_start_loc, req_slice)

    expected = torch.tensor([0, 5, 12])
    assert torch.equal(result, expected)
```
**EN:** Test case covering `slice from beginning`. Inputs/fixtures: `sample_query_start_loc`. It exercises `slice, slice_query_start_locs, torch.tensor, torch.equal`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `slice from beginning` 的测试用例。 输入或 fixture：`sample_query_start_loc`。 该测试会调用 `slice, slice_query_start_locs, torch.tensor, torch.equal`。 代码主体包含 1 个显式断言。

### test_slice_to_end (lines 45-51)
```python
def test_slice_to_end(sample_query_start_loc):
    """Test slicing to the end of tensor"""
    req_slice = slice(3, 5)  # slice from index 3 to 5 (last index)
    result = slice_query_start_locs(sample_query_start_loc, req_slice)

    expected = torch.tensor([0, 15, 30])
    assert torch.equal(result, expected)
```
**EN:** Test case covering `slice to end`. Inputs/fixtures: `sample_query_start_loc`. It exercises `slice, slice_query_start_locs, torch.tensor, torch.equal`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `slice to end` 的测试用例。 输入或 fixture：`sample_query_start_loc`。 该测试会调用 `slice, slice_query_start_locs, torch.tensor, torch.equal`。 代码主体包含 1 个显式断言。

### test_single_element_slice (lines 54-60)
```python
def test_single_element_slice(sample_query_start_loc):
    """Test slice that results in single element"""
    req_slice = slice(2, 3)  # slice from index 2 to 3
    result = slice_query_start_locs(sample_query_start_loc, req_slice)

    expected = torch.tensor([0, 8])
    assert torch.equal(result, expected)
```
**EN:** Test case covering `single element slice`. Inputs/fixtures: `sample_query_start_loc`. It exercises `slice, slice_query_start_locs, torch.tensor, torch.equal`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `single element slice` 的测试用例。 输入或 fixture：`sample_query_start_loc`。 该测试会调用 `slice, slice_query_start_locs, torch.tensor, torch.equal`。 代码主体包含 1 个显式断言。

### test_full_tensor_slice (lines 63-69)
```python
def test_full_tensor_slice(sample_query_start_loc):
    """Test slicing the entire tensor"""
    req_slice = slice(0, 5)  # slice entire tensor
    result = slice_query_start_locs(sample_query_start_loc, req_slice)

    expected = torch.tensor([0, 5, 12, 20, 35, 50])
    assert torch.equal(result, expected)
```
**EN:** Test case covering `full tensor slice`. Inputs/fixtures: `sample_query_start_loc`. It exercises `slice, slice_query_start_locs, torch.tensor, torch.equal`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `full tensor slice` 的测试用例。 输入或 fixture：`sample_query_start_loc`。 该测试会调用 `slice, slice_query_start_locs, torch.tensor, torch.equal`。 代码主体包含 1 个显式断言。

### test_slice_bounds_edge_cases (lines 72-78)
```python
def test_slice_bounds_edge_cases(sample_query_start_loc):
    # Test slice that goes exactly to the last element
    req_slice = slice(4, 5)  # Last index
    result = slice_query_start_locs(sample_query_start_loc, req_slice)

    expected = torch.tensor([0, 15])
    assert torch.equal(result, expected)
```
**EN:** Test case covering `slice bounds edge cases`. Inputs/fixtures: `sample_query_start_loc`. It exercises `slice, slice_query_start_locs, torch.tensor, torch.equal`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `slice bounds edge cases` 的测试用例。 输入或 fixture：`sample_query_start_loc`。 该测试会调用 `slice, slice_query_start_locs, torch.tensor, torch.equal`。 代码主体包含 1 个显式断言。

### small_decode_metadata (lines 82-86)
```python
def small_decode_metadata():
    """Create metadata for small decode batch"""
    batch_spec = BATCH_SPECS["small_decode"]
    device = torch.device("cpu")
    return create_common_attn_metadata(batch_spec, block_size=16, device=device)
```
**EN:** Fixture/helper `small_decode_metadata` prepares reusable state for downstream tests. Key calls include `torch.device, create_common_attn_metadata`.
**CN:** `small_decode_metadata` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `torch.device, create_common_attn_metadata`。

### large_decode_metadata (lines 90-94)
```python
def large_decode_metadata():
    """Create metadata for small decode batch"""
    batch_spec = BATCH_SPECS["large_decode"]
    device = torch.device("cpu")
    return create_common_attn_metadata(batch_spec, block_size=16, device=device)
```
**EN:** Fixture/helper `large_decode_metadata` prepares reusable state for downstream tests. Key calls include `torch.device, create_common_attn_metadata`.
**CN:** `large_decode_metadata` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `torch.device, create_common_attn_metadata`。

### mixed_small_metadata (lines 98-102)
```python
def mixed_small_metadata():
    """Create metadata for mixed small batch"""
    batch_spec = BATCH_SPECS["mixed_small"]
    device = torch.device("cpu")
    return create_common_attn_metadata(batch_spec, block_size=16, device=device)
```
**EN:** Fixture/helper `mixed_small_metadata` prepares reusable state for downstream tests. Key calls include `torch.device, create_common_attn_metadata`.
**CN:** `mixed_small_metadata` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `torch.device, create_common_attn_metadata`。

### test_make_metadata_with_slice_decode_batch (lines 106-118)
```python
def test_make_metadata_with_slice_decode_batch(small_decode_metadata):
    """Test slicing decode batch metadata"""
    # Split first request only
    ubatch_slice = UBatchSlice(slice(0, 1), slice(0, 1))

    result = _make_metadata_with_slice(ubatch_slice, small_decode_metadata)

    # Check sliced results
    assert result.num_reqs == 1  # slice(0, 1) gives 1 requests
    assert result.num_actual_tokens == 1  # slice(0, 1) gives 1 token
    assert result.max_query_len == 1
    assert torch.equal(result.query_start_loc, torch.tensor([0, 1]))
    assert torch.equal(result.seq_lens, torch.tensor([32]))
```
**EN:** Test case covering `make metadata with slice decode batch`. Inputs/fixtures: `small_decode_metadata`. It exercises `UBatchSlice, _make_metadata_with_slice, torch.equal, slice, torch.tensor`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `make metadata with slice decode 批处理` 的测试用例。 输入或 fixture：`small_decode_metadata`。 该测试会调用 `UBatchSlice, _make_metadata_with_slice, torch.equal, slice, torch.tensor`。 代码主体包含 5 个显式断言。

### test_make_metadata_with_slice_mixed_batch (lines 121-131)
```python
def test_make_metadata_with_slice_mixed_batch(mixed_small_metadata):
    """Test slicing mixed batch metadata"""
    ubatch_slice = UBatchSlice(slice(1, 3), slice(1, 7))  # Requests 1-3, tokens 1-7

    result = _make_metadata_with_slice(ubatch_slice, mixed_small_metadata)

    assert result.num_reqs == 2  # slice(1, 3) gives 2 requests
    assert result.num_actual_tokens == 6  # slice(1, 7) gives 6 tokens
    assert result.max_query_len == 5
    assert torch.equal(result.query_start_loc, torch.tensor([0, 1, 6]))
    assert torch.equal(result.seq_lens, torch.tensor([40, 48]))
```
**EN:** Test case covering `make metadata with slice mixed batch`. Inputs/fixtures: `mixed_small_metadata`. It exercises `UBatchSlice, _make_metadata_with_slice, torch.equal, slice, torch.tensor`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `make metadata with slice mixed 批处理` 的测试用例。 输入或 fixture：`mixed_small_metadata`。 该测试会调用 `UBatchSlice, _make_metadata_with_slice, torch.equal, slice, torch.tensor`。 代码主体包含 5 个显式断言。

### test_split_attn_metadata_decode_batch (lines 134-155)
```python
def test_split_attn_metadata_decode_batch(large_decode_metadata):
    """Test splitting decode batch into two equal parts"""
    num_tokens = large_decode_metadata.num_reqs
    mid_point = num_tokens // 2
    ubatch_slices = [
        UBatchSlice(slice(0, mid_point), slice(0, mid_point)),
        UBatchSlice(slice(mid_point, num_tokens), slice(mid_point, num_tokens)),
    ]

    results = split_attn_metadata(ubatch_slices, large_decode_metadata)

    assert len(results) == 2

    # Check first split
    assert results[0].num_reqs == mid_point
    assert results[0].num_actual_tokens == mid_point
    assert torch.equal(results[0].seq_lens, torch.tensor([2048] * mid_point))

    # Check second split
    assert results[1].num_reqs == mid_point
    assert results[1].num_actual_tokens == mid_point
    assert torch.equal(results[1].seq_lens, torch.tensor([2048] * mid_point))
```
**EN:** Test case covering `split attn metadata decode batch`. Inputs/fixtures: `large_decode_metadata`. It exercises `split_attn_metadata, torch.equal, UBatchSlice, len, torch.tensor, slice`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `split attn metadata decode 批处理` 的测试用例。 输入或 fixture：`large_decode_metadata`。 该测试会调用 `split_attn_metadata, torch.equal, UBatchSlice, len, torch.tensor, slice`。 代码主体包含 7 个显式断言。

### apply_split_decodes_and_prefills (lines 158-181)
```python
def apply_split_decodes_and_prefills(
    query_lens: list[int],
    decode_threshold: int,
    require_uniform: bool,
    padded_num_tokens: int | None = None,
):
    """Helper function to apply split_decodes_and_prefills and return
    the results."""
    device = torch.device("cpu")
    seq_lens = [10 * (i + 1) for i in range(len(query_lens))]
    common_metadata = create_common_attn_metadata(
        BatchSpec(seq_lens=seq_lens, query_lens=query_lens),
        block_size=16,
        device=device,
    )

    if padded_num_tokens is not None:
        common_metadata.num_actual_tokens = padded_num_tokens

    return split_decodes_and_prefills(
        common_metadata,
        decode_threshold=decode_threshold,
        require_uniform=require_uniform,
    )
```
**EN:** Helper function `apply_split_decodes_and_prefills` encapsulates reusable logic for `apply split decodes and prefills`. Inputs: `query_lens, decode_threshold, require_uniform, padded_num_tokens`. Key calls include `torch.device, create_common_attn_metadata, split_decodes_and_prefills, BatchSpec, range, len`.
**CN:** 辅助函数 `apply_split_decodes_and_prefills` 封装了与 `apply split decodes and prefills` 相关的可复用逻辑。 输入参数：`query_lens, decode_threshold, require_uniform, padded_num_tokens`。 关键调用包括 `torch.device, create_common_attn_metadata, split_decodes_and_prefills, BatchSpec, range, len`。

### test_split_decodes_and_prefills_nonuniform_all_ones (lines 184-192)
```python
def test_split_decodes_and_prefills_nonuniform_all_ones():
    query_lens = [1, 1, 1]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 1, False)
    )
    assert num_decodes == 3
    assert num_prefills == 0
    assert num_decode_tokens == 3
    assert num_prefill_tokens == 0
```
**EN:** Test case covering `split decodes and prefills nonuniform all ones`. It exercises `apply_split_decodes_and_prefills`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills nonuniform all ones` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_nonuniform_all_short_decodes (lines 195-203)
```python
def test_split_decodes_and_prefills_nonuniform_all_short_decodes():
    query_lens = [1, 2, 1, 3, 2, 1, 2]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 3, False)
    )
    assert num_decodes == 7
    assert num_prefills == 0
    assert num_decode_tokens == sum(query_lens)
    assert num_prefill_tokens == 0
```
**EN:** Test case covering `split decodes and prefills nonuniform all short decodes`. It exercises `apply_split_decodes_and_prefills, sum`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills nonuniform all short decodes` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills, sum`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_nonuniform_all_prefills (lines 206-214)
```python
def test_split_decodes_and_prefills_nonuniform_all_prefills():
    query_lens = [4, 5, 6, 7]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 3, False)
    )
    assert num_decodes == 0
    assert num_prefills == 4
    assert num_decode_tokens == 0
    assert num_prefill_tokens == sum(query_lens)
```
**EN:** Test case covering `split decodes and prefills nonuniform all prefills`. It exercises `apply_split_decodes_and_prefills, sum`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills nonuniform all prefills` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills, sum`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_nonuniform_mixed_batch (lines 217-225)
```python
def test_split_decodes_and_prefills_nonuniform_mixed_batch():
    query_lens = [2, 1, 3, 4, 5, 6, 7, 8]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 4, False)
    )
    assert num_decodes == 4  # 2, 1, 3, 4 are all <= 4
    assert num_prefills == 4  # 5, 6, 7, 8 are all > 4
    assert num_decode_tokens == 10  # 2 + 1 + 3 + 4
    assert num_prefill_tokens == 26  # 5 + 6 + 7 + 8
```
**EN:** Test case covering `split decodes and prefills nonuniform mixed batch`. It exercises `apply_split_decodes_and_prefills`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills nonuniform mixed 批处理` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_uniform_all_ones (lines 228-236)
```python
def test_split_decodes_and_prefills_uniform_all_ones():
    query_lens = [1, 1, 1]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 1, True)
    )
    assert num_decodes == 3
    assert num_prefills == 0
    assert num_decode_tokens == 3
    assert num_prefill_tokens == 0
```
**EN:** Test case covering `split decodes and prefills uniform all ones`. It exercises `apply_split_decodes_and_prefills`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills uniform all ones` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_uniform_all_short_decodes (lines 239-247)
```python
def test_split_decodes_and_prefills_uniform_all_short_decodes():
    query_lens = [2, 2, 1, 3, 2, 1, 2]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 3, True)
    )
    assert num_decodes == 2
    assert num_prefills == 5
    assert num_decode_tokens == 4
    assert num_prefill_tokens == (1 + 3 + 2 + 1 + 2)
```
**EN:** Test case covering `split decodes and prefills uniform all short decodes`. It exercises `apply_split_decodes_and_prefills`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills uniform all short decodes` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_uniform_all_prefills (lines 250-258)
```python
def test_split_decodes_and_prefills_uniform_all_prefills():
    query_lens = [4, 5, 6, 7]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 3, True)
    )
    assert num_decodes == 0
    assert num_prefills == 4
    assert num_decode_tokens == 0
    assert num_prefill_tokens == sum(query_lens)
```
**EN:** Test case covering `split decodes and prefills uniform all prefills`. It exercises `apply_split_decodes_and_prefills, sum`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills uniform all prefills` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills, sum`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_uniform_mixed_batch_all_uniform_decodes (lines 261-269)
```python
def test_split_decodes_and_prefills_uniform_mixed_batch_all_uniform_decodes():
    query_lens = [2, 2, 2, 4, 5, 6, 7, 8]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 4, True)
    )
    assert num_decodes == 3  # 2, 2, 2 are all <= 4 and uniform
    assert num_prefills == 5  # 4, 5, 6, 7, 8 are all > 4
    assert num_decode_tokens == 6  # 2 + 2 + 2
    assert num_prefill_tokens == 30  # 4 + 5 + 6 + 7 + 8
```
**EN:** Test case covering `split decodes and prefills uniform mixed batch all uniform decodes`. It exercises `apply_split_decodes_and_prefills`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills uniform mixed 批处理 all uniform decodes` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_uniform_mixed_batch_non_uniform_decodes (lines 272-280)
```python
def test_split_decodes_and_prefills_uniform_mixed_batch_non_uniform_decodes():
    query_lens = [2, 1, 2, 4, 5, 6, 7, 8]
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 4, True)
    )
    assert num_decodes == 1  # only the first 2 is taken as decode
    assert num_prefills == 7  # 1, 2, 4, 5, 6, 7, 8 are all > 4 or non-uniform
    assert num_decode_tokens == 2  # only the first 2
    assert num_prefill_tokens == (sum(query_lens) - 2)  # rest of the tokens
```
**EN:** Test case covering `split decodes and prefills uniform mixed batch non uniform decodes`. It exercises `apply_split_decodes_and_prefills, sum`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills uniform mixed 批处理 non uniform decodes` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills, sum`。 代码主体包含 4 个显式断言。

### test_split_decodes_and_prefills_uniform_padded_batch_all_same (lines 283-296)
```python
def test_split_decodes_and_prefills_uniform_padded_batch_all_same():
    """uniform batch where all query lengths are identical with 0 length padded reqs."""
    # All query lengths are 2, with decode_threshold=3 (so 2 <= 3)
    # This triggers the padded uniform path at line 891
    query_lens = [2, 2, 2, 0]
    padded_num_tokens = 8
    num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
        apply_split_decodes_and_prefills(query_lens, 3, True, padded_num_tokens)
    )
    # With uniform batch, all requests are treated as decodes
    assert num_decodes == 4
    assert num_prefills == 0
    assert num_decode_tokens == padded_num_tokens
    assert num_prefill_tokens == 0
```
**EN:** Test case covering `split decodes and prefills uniform padded batch all same`. It exercises `apply_split_decodes_and_prefills`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `split decodes and prefills uniform padded 批处理 all same` 的测试用例。 该测试会调用 `apply_split_decodes_and_prefills`。 代码主体包含 4 个显式断言。

### test_prefill_split_across_ubatches (lines 308-381)
```python
def test_prefill_split_across_ubatches(
    seq_lens, query_lens, split_point, expected_first_reqs, expected_second_reqs
):
    """Test splitting a prefill across ubatches"""
    import numpy as np

    device = torch.device("cpu")
    batch_spec = BatchSpec(seq_lens=seq_lens, query_lens=query_lens)
    common = create_common_attn_metadata(batch_spec, block_size=16, device=device)
    num_scheduled_tokens = np.array(query_lens, dtype=np.int32)
    qsl_np = common.query_start_loc_cpu.numpy()
    num_tokens = common.num_actual_tokens
    ubatch_slices, _ = maybe_create_ubatch_slices(
        True,
        num_scheduled_tokens,
        num_tokens,
    # ... excerpt omitted for brevity ...
    assert ubatch_slices is not None and len(ubatch_slices) == 2
    assert first_meta.num_actual_tokens == split_point
    assert second_meta.num_actual_tokens == num_tokens - split_point
    assert first_meta.num_reqs == expected_first_reqs
    assert second_meta.num_reqs == expected_second_reqs
    assert qlen_first_last == tokens_in_first_chunk
    assert int(second_meta.seq_lens[0]) == seq_lens[split_req_idx]
    # Any following full requests in second ubatch should match originals
    for j in range(1, second_meta.num_reqs):
        # Map to original request index
        orig_idx = split_req_idx + j
        assert int(second_meta.seq_lens[j]) == seq_lens[orig_idx]
```
**EN:** Parameterized test covering `prefill split across ubatches`. Parameter axes: `seq_lens, query_lens, split_point, expected_first_reqs, expected_second_reqs`. Inputs/fixtures: `seq_lens, query_lens, split_point, expected_first_reqs, expected_second_reqs`. It exercises `mark.parametrize, torch.device, BatchSpec, create_common_attn_metadata, np.array, query_start_loc_cpu.numpy`. The body contains 11 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill split across ubatches` 的测试用例。 参数维度：`seq_lens, query_lens, split_point, expected_first_reqs, expected_second_reqs`。 输入或 fixture：`seq_lens, query_lens, split_point, expected_first_reqs, expected_second_reqs`。 该测试会调用 `mark.parametrize, torch.device, BatchSpec, create_common_attn_metadata, np.array, query_start_loc_cpu.numpy`。 代码主体包含 11 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Worker-side execution and device coordination
- **CN:** worker 侧执行与设备协同

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, numpy`.
- **CN:** 外部库：`pytest, torch, numpy`。
- **EN:** vLLM modules under test: `vllm.v1.attention.backends.utils, vllm.v1.worker.ubatch_utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.attention.backends.utils, vllm.v1.worker.ubatch_utils`。
- **EN:** Local test helpers: `tests.v1.attention.test_attention_backends, tests.v1.attention.utils`.
- **CN:** 本地测试辅助模块：`tests.v1.attention.test_attention_backends, tests.v1.attention.utils`。
