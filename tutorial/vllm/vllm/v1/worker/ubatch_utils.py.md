# ubatch_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/ubatch_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `UBatchSlice`, `is_last_ubatch_empty`, `check_ubatch_thresholds` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `UBatchSlice`, `is_last_ubatch_empty`, `check_ubatch_thresholds`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from dataclasses import dataclass
from typing import TypeAlias

import numpy as np
import torch

from vllm.config import ParallelConfig
from vllm.v1.attention.backend import CommonAttentionMetadata
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.config`, `vllm.v1.attention.backend`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.config`, `vllm.v1.attention.backend` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `UBatchSlice` class / `UBatchSlice` 类
```python
@dataclass
class UBatchSlice:
    request_slice: slice
    token_slice: slice
```
**EN:** Uses `@dataclass` to package related state for `UBatchSlice`. Typical fields include `request_slice`, `token_slice`.
**CN:** `UBatchSlice` 使用 `@dataclass` 打包相关状态。典型字段包括 `request_slice`, `token_slice`。

### `UBatchSlice.is_empty` method / `UBatchSlice.is_empty` 方法
```python
    def is_empty(self) -> bool:
        return (
            self.request_slice.start == self.request_slice.stop
            or self.token_slice.start == self.token_slice.stop
        )
```
**EN:** This method answers a boolean capability check within `UBatchSlice`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`UBatchSlice`。

### `UBatchSlice.num_tokens` method / `UBatchSlice.num_tokens` 方法
```python
    @property
    def num_tokens(self) -> int:
        return self.token_slice.stop - self.token_slice.start
```
**EN:** This method implements `num_tokens` within `UBatchSlice`.
**CN:** 该方法会实现 `num_tokens`，其作用域位于`UBatchSlice`。

### Module constants / 模块常量
```python
UBatchSlices: TypeAlias = list[UBatchSlice]
```
**EN:** Defines module-level constants or aliases such as `UBatchSlices`, which are reused by later definitions.
**CN:** 定义 `UBatchSlices` 等模块级常量或别名，供后续定义复用。

### `is_last_ubatch_empty` function / `is_last_ubatch_empty` 函数
```python
def is_last_ubatch_empty(
    orig_num_tokens: int, padded_num_tokens: int, num_ubatches: int
) -> bool:
    return (padded_num_tokens // num_ubatches) * (num_ubatches - 1) >= orig_num_tokens
```
**EN:** This function answers a boolean capability check within the module.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。

### `check_ubatch_thresholds` function / `check_ubatch_thresholds` 函数
```python
def check_ubatch_thresholds(
    config: ParallelConfig, num_tokens: int, uniform_decode: bool
) -> bool:
    if not config.use_ubatching:
        return False
    if uniform_decode:
        return num_tokens >= config.dbo_decode_token_threshold
    else:
        return num_tokens >= config.dbo_prefill_token_threshold
```
**EN:** This function validates assumptions or constraints within the module. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_pad_out_ubatch_slices` function / `_pad_out_ubatch_slices` 函数
```python
def _pad_out_ubatch_slices(
    ubatch_slices: UBatchSlices, num_total_tokens: int, num_reqs_padded: int
) -> UBatchSlices:
    last_slice = ubatch_slices[-1]
    padded_last_request_slice = slice(last_slice.request_slice.start, num_reqs_padded)
    padded_last_token_slice = slice(last_slice.token_slice.start, num_total_tokens)

    return ubatch_slices[:-1] + [
        UBatchSlice(padded_last_request_slice, padded_last_token_slice)
    ]
```
**EN:** This function implements `_pad_out_ubatch_slices` within the module. Key calls include `slice`, `UBatchSlice`.
**CN:** 该函数会实现 `_pad_out_ubatch_slices`，其作用域位于the module。 关键调用包括 `slice`, `UBatchSlice`。

### `maybe_create_ubatch_slices` function / `maybe_create_ubatch_slices` 函数
```python
def maybe_create_ubatch_slices(
    should_ubatch: bool,
    num_scheduled_tokens: np.ndarray,
    num_tokens_padded: int,
    num_reqs_padded: int,
    num_ubatches: int,
    split_point: list[int] | int | None = None,
) -> tuple[UBatchSlices | None, UBatchSlices | None]:
    if not should_ubatch:
        return None, None

    if split_point is None:
        split_point = int(num_tokens_padded) // num_ubatches

    token_split_points = [split_point * i for i in range(1, num_ubatches)]

    # TODO(lucas): Refactor the gpu_model_runner.py so we can pass
    # in cu_num_tokens directly (i.e. query_start_loc)
    cu_num_tokens = np.zeros(len(num_scheduled_tokens) + 1, dtype=np.int32)
    np.cumsum(num_scheduled_tokens, dtype=np.int32, out=cu_num_tokens[1:])

    ubatch_slices = []
    start_token = 0

    # Add the end point to the split points to make iteration easier
    all_points = token_split_points + [cu_num_tokens[-1]]

    for end_token in all_points:
        token_slice = slice(start_token, end_token)

        # Determine request slices using exclusive stop semantics
        # Ubatch includes requests whose tokens overlap [start_token, end_token)

        # Start at the request that contains the start_token
        # or the request starting exactly at start_token (if on boundary)
        req_start = int(np.searchsorted(cu_num_tokens, start_token, side="right") - 1)

        # Stop at the request that starts at or after end_token
        req_stop = int(np.searchsorted(cu_num_tokens, end_token, side="left"))

        req_slice = slice(req_start, req_stop)
        ubatch_slices.append(UBatchSlice(req_slice, token_slice))

        start_token = end_token

    ubatch_slices_padded = _pad_out_ubatch_slices(
        ubatch_slices, num_tokens_padded, num_reqs_padded
    )

    assert sum(s.num_tokens for s in ubatch_slices_padded) == num_tokens_padded

    return ubatch_slices, ubatch_slices_padded
```
**EN:** This function applies logic conditionally within the module. Key calls include `zeros`, `cumsum`, `_pad_out_ubatch_slices`, `slice`, `int`, `append`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会按条件执行逻辑，其作用域位于the module。 关键调用包括 `zeros`, `cumsum`, `_pad_out_ubatch_slices`, `slice`, `int`, `append`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `slice_query_start_locs` function / `slice_query_start_locs` 函数
```python
def slice_query_start_locs(
    query_start_loc: torch.Tensor,
    request_slice: slice,
) -> torch.Tensor:
    """
    Creates a new query_start_loc that corresponds to the requests in
    request_slice.

    Note: This function creates a new tensor to hold the new query_start_locs.
    This will break cudagraph compatibility.
    """
    return (
        query_start_loc[request_slice.start : request_slice.stop + 1]
        - query_start_loc[request_slice.start]
    )
```
**EN:** This function implements `slice_query_start_locs` within the module. The docstring frames it as: Creates a new query_start_loc that corresponds to the requests in request_slice.
**CN:** 该函数会实现 `slice_query_start_locs`，其作用域位于the module。

### `_make_metadata_with_slice` function / `_make_metadata_with_slice` 函数
```python
def _make_metadata_with_slice(
    ubatch_slice: UBatchSlice, attn_metadata: CommonAttentionMetadata
) -> CommonAttentionMetadata:
    """
    This function creates a new CommonAttentionMetadata that corresponds to
    the requests included in ubatch_slice
    """

    assert not ubatch_slice.is_empty(), f"Ubatch slice {ubatch_slice} is empty"

    request_slice = ubatch_slice.request_slice
    token_slice = ubatch_slice.token_slice

    start_locs = attn_metadata.query_start_loc_cpu
    first_req = request_slice.start
    first_tok = token_slice.start
    last_req = request_slice.stop - 1
    last_tok = token_slice.stop - 1

    assert start_locs[first_req] <= first_tok < start_locs[first_req + 1], (
        "Token slice start outside of first request"
    )
    # NOTE: last token can be outside of the last request if we have CG padding.

    # If the request is split across ubatches, we have to adjust the metadata.
    # splits_first_request: The first request in this slice is the continuation of
    #                       a request that started in a previous slice.
    # splits_last_request:  The last request in this slice continues into the
    #                       next slice.
    splits_first_request = first_tok > start_locs[first_req]
    splits_last_request = last_tok < start_locs[last_req + 1] - 1

    query_start_loc_cpu = slice_query_start_locs(start_locs, request_slice)
    query_start_loc = slice_query_start_locs(
        attn_metadata.query_start_loc, request_slice
    )

    assert len(query_start_loc) >= 2, (
        f"query_start_loc must have at least 2 elements, got {len(query_start_loc)}"
    )

    if splits_first_request:
        tokens_skipped = first_tok - start_locs[first_req]
        query_start_loc[1:] -= tokens_skipped
        query_start_loc_cpu[1:] -= tokens_skipped
    seq_lens = attn_metadata.seq_lens[request_slice]
    # Read raw fields to avoid triggering the deprecated D2H-syncing properties.
    seq_lens_cpu = (
        attn_metadata._seq_lens_cpu[request_slice]
        if attn_metadata._seq_lens_cpu is not None
# ... omitted for brevity ...
        torch.max(torch.abs(query_start_loc_cpu[1:] - query_start_loc_cpu[:-1])).item()
    )

    # This is to account for the case where we are in a dummy
    # run and query_start_loc_cpu is full of 0s
    if max_query_len == 0:
        max_query_len = attn_metadata.max_query_len

    block_table_tensor = attn_metadata.block_table_tensor[request_slice]
    slot_mapping = attn_metadata.slot_mapping[token_slice]

    return CommonAttentionMetadata(
        query_start_loc=query_start_loc,
        query_start_loc_cpu=query_start_loc_cpu,
        seq_lens=seq_lens,
        num_reqs=num_requests,
        num_actual_tokens=num_actual_tokens,
        max_query_len=max_query_len,
        max_seq_len=max_seq_len,
        block_table_tensor=block_table_tensor,
        slot_mapping=slot_mapping,
        seq_lens_cpu_upper_bound=seq_lens_cpu_upper_bound,
        _seq_lens_cpu=seq_lens_cpu,
        _num_computed_tokens_cpu=num_computed_tokens_cpu,
    )
```
**EN:** This function implements `_make_metadata_with_slice` within the module. The docstring frames it as: This function creates a new CommonAttentionMetadata that corresponds to the requests included in ubatch_slice Key calls include `slice_query_start_locs`, `max`, `int`, `CommonAttentionMetadata`, `is_empty`, `len`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_make_metadata_with_slice`，其作用域位于the module。 关键调用包括 `slice_query_start_locs`, `max`, `int`, `CommonAttentionMetadata`, `is_empty`, `len`。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `split_attn_metadata` function / `split_attn_metadata` 函数
```python
def split_attn_metadata(
    ubatch_slices: list[UBatchSlice],
    common_attn_metadata: CommonAttentionMetadata,
) -> list[CommonAttentionMetadata]:
    """
    Creates a new CommonAttentionMetadata instance that corresponds to the
    requests for each UBatchSlice in ubatch_slices.

    Note: This function does not modify common_attn_metadata
    """
    results = []
    for ubatch_slice in ubatch_slices:
        results.append(_make_metadata_with_slice(ubatch_slice, common_attn_metadata))

    return results
```
**EN:** This function implements `split_attn_metadata` within the module. The docstring frames it as: Creates a new CommonAttentionMetadata instance that corresponds to the requests for each UBatchSlice in ubatch_slices. Key calls include `append`, `_make_metadata_with_slice`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `split_attn_metadata`，其作用域位于the module。 关键调用包括 `append`, `_make_metadata_with_slice`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `UBatchSlice`: central class or interface in this module. / `UBatchSlice`：本模块中的核心类或接口。
- `is_last_ubatch_empty`: top-level helper or orchestration entry point. / `is_last_ubatch_empty`：顶层辅助函数或编排入口。
- `check_ubatch_thresholds`: top-level helper or orchestration entry point. / `check_ubatch_thresholds`：顶层辅助函数或编排入口。
- `_pad_out_ubatch_slices`: top-level helper or orchestration entry point. / `_pad_out_ubatch_slices`：顶层辅助函数或编排入口。
- `maybe_create_ubatch_slices`: top-level helper or orchestration entry point. / `maybe_create_ubatch_slices`：顶层辅助函数或编排入口。
- `slice_query_start_locs`: top-level helper or orchestration entry point. / `slice_query_start_locs`：顶层辅助函数或编排入口。
- `_make_metadata_with_slice`: top-level helper or orchestration entry point. / `_make_metadata_with_slice`：顶层辅助函数或编排入口。
- `split_attn_metadata`: top-level helper or orchestration entry point. / `split_attn_metadata`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.v1.attention.backend`
