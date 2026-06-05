# logprob.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/logprob.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_topk_log_softmax_kernel`, `_ranks_kernel`, `compute_token_logprobs` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `_topk_log_softmax_kernel`, `_ranks_kernel`, `compute_token_logprobs`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import numpy as np
import torch

from vllm.sampling_params import MAX_LOGPROB_TOKEN_IDS, SamplingParams
from vllm.triton_utils import tl, triton
from vllm.v1.outputs import LogprobsTensors
from vllm.v1.worker.gpu.buffer_utils import StagedWriteTensor, UvaBackedTensor
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.sampling_params`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.worker.gpu.buffer_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.sampling_params`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.worker.gpu.buffer_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_topk_log_softmax_kernel` function / `_topk_log_softmax_kernel` 函数
```python
@triton.jit
def _topk_log_softmax_kernel(
    output_ptr,
    logits_ptr,
    logits_stride,
    topk_ids_ptr,
    topk,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
    PADDED_TOPK: tl.constexpr,
):
    req_idx = tl.program_id(0)
    row_ptr = logits_ptr + req_idx * logits_stride

    max_val = float("-inf")
    for i in range(0, vocab_size, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        logits = tl.load(row_ptr + block, mask=block < vocab_size, other=float("-inf"))
        max_val = tl.max(tl.maximum(logits, max_val))
    max_val = max_val.to(tl.float32)  # type: ignore

    se = 0.0
    for i in range(0, vocab_size, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        logits = tl.load(row_ptr + block, mask=block < vocab_size, other=0.0)
        # NOTE(woosuk): Make sure that logits and all following operations use FP32.
        logits = logits.to(tl.float32)
        e = tl.exp(logits - max_val)
        e = tl.where(block < vocab_size, e, 0.0)
        se += tl.sum(e)
    lse = tl.log(se)

    k_offset = tl.arange(0, PADDED_TOPK)
    k_mask = k_offset < topk
    topk_ids = tl.load(topk_ids_ptr + req_idx * topk + k_offset, mask=k_mask, other=0)

    logits = tl.load(row_ptr + topk_ids, mask=k_mask)
    logits = logits.to(tl.float32)
    o = logits - max_val - lse
    tl.store(output_ptr + req_idx * topk + k_offset, o, mask=k_mask)
```
**EN:** This function implements `_topk_log_softmax_kernel` within the module. Key calls include `program_id`, `float`, `range`, `to`, `log`, `arange`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_topk_log_softmax_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `float`, `range`, `to`, `log`, `arange`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `_ranks_kernel` function / `_ranks_kernel` 函数
```python
@triton.jit
def _ranks_kernel(
    output_ptr,
    logits_ptr,
    logits_stride,
    token_ids_ptr,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
):
    req_idx = tl.program_id(0)
    row_ptr = logits_ptr + req_idx * logits_stride

    token_id = tl.load(token_ids_ptr + req_idx)
    x = tl.load(row_ptr + token_id)

    n = 0
    for i in range(0, vocab_size, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        logits = tl.load(row_ptr + block, mask=block < vocab_size, other=float("-inf"))
        n += tl.sum((logits >= x).to(tl.int32))
    tl.store(output_ptr + req_idx, n)
```
**EN:** This function implements `_ranks_kernel` within the module. Key calls include `program_id`, `load`, `range`, `store`, `sum`, `arange`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_ranks_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `range`, `store`, `sum`, `arange`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `compute_token_logprobs` function / `compute_token_logprobs` 函数
```python
def compute_token_logprobs(
    logits: torch.Tensor, token_ids: torch.Tensor
) -> torch.Tensor:
    # NOTE(woosuk): To save GPU memory, we do not materialize the full
    # [batch_size, vocab_size] logprobs tensor. The kernel computes
    # max + logsumexp per row and only emits logprobs at `token_ids`.
    batch_size, vocab_size = logits.shape
    token_ids = token_ids.to(torch.int64)
    num_logprobs = token_ids.shape[1]
    logprobs = logits.new_empty((batch_size, num_logprobs), dtype=torch.float32)
    _topk_log_softmax_kernel[(batch_size,)](
        logprobs,
        logits,
        logits.stride(0),
        token_ids,
        num_logprobs,
        vocab_size,
        BLOCK_SIZE=1024,  # type: ignore
        PADDED_TOPK=triton.next_power_of_2(num_logprobs),
    )
    return logprobs
```
**EN:** This function computes derived values within the module. Key calls include `to`, `new_empty`, `stride`, `next_power_of_2`.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `to`, `new_empty`, `stride`, `next_power_of_2`。

### `compute_topk_logprobs` function / `compute_topk_logprobs` 函数
```python
def compute_topk_logprobs(
    logits: torch.Tensor,
    num_logprobs: int,
    sampled_token_ids: torch.Tensor,
    cu_num_logits: list[int] | None = None,
    logprob_token_ids_state: "LogprobTokenIdsState | None" = None,
    expanded_idx_mapping: torch.Tensor | None = None,
    max_per_req_token_ids: int = 0,
) -> LogprobsTensors:
    assert num_logprobs >= 0
    batch_size, vocab_size = logits.shape

    if max_per_req_token_ids == 0:
        # Fast path: no request asked for custom logprob_token_ids.
        logprob_token_ids = sampled_token_ids.unsqueeze(-1)
        if num_logprobs > 0:
            topk_indices = torch.topk(logits, num_logprobs, dim=-1).indices
            logprob_token_ids = torch.cat((logprob_token_ids, topk_indices), dim=1)
        logprobs = compute_token_logprobs(logits, logprob_token_ids)
    else:
        # Some requests specified logprob_token_ids. Build the [batch_size,
        # 1 + max_cols] token_ids matrix and validity mask on the GPU via a
        # single triton kernel, overriding the topk columns with per-request
        # tokens where applicable.
        assert logprob_token_ids_state is not None
        assert expanded_idx_mapping is not None

        if num_logprobs > 0:
            topk_token_ids = torch.topk(logits, num_logprobs, dim=-1).indices
            topk_token_ids = topk_token_ids.to(torch.int32)
        else:
            # This tensor just used as an int32 pointer, data not accessed.
            topk_token_ids = logprob_token_ids_state.token_ids.gpu

        num_cols = max(num_logprobs, max_per_req_token_ids)
        logprob_token_ids = sampled_token_ids.new_zeros((batch_size, 1 + num_cols))
        valid_mask = torch.zeros_like(logprob_token_ids, dtype=torch.bool)
        _fill_logprob_token_ids_kernel[(batch_size,)](
            logprob_token_ids,
            logprob_token_ids.stride(0),
            valid_mask,
            valid_mask.stride(0),
            sampled_token_ids,
            topk_token_ids,
            topk_token_ids.stride(0),
            expanded_idx_mapping,
            logprob_token_ids_state.num_token_ids.gpu,
            logprob_token_ids_state.token_ids.gpu,
            logprob_token_ids_state.token_ids.gpu.stride(0),
            NUM_TOPK=num_logprobs,
            PADDED_COLS=triton.next_power_of_2(num_cols),
        )
        logprobs = compute_token_logprobs(logits, logprob_token_ids)
        logprobs = logprobs.masked_fill(~valid_mask, float("-inf"))

    token_ranks = torch.empty(batch_size, dtype=torch.int64, device=logits.device)
    _ranks_kernel[(batch_size,)](
        token_ranks,
        logits,
        logits.stride(0),
        sampled_token_ids,
        vocab_size,
        BLOCK_SIZE=8192,  # type: ignore
    )
    return LogprobsTensors(
        logprob_token_ids=logprob_token_ids,
        logprobs=logprobs,
        selected_token_ranks=token_ranks,
        cu_num_generated_tokens=cu_num_logits,
    )
```
**EN:** This function computes derived values within the module. Key calls include `empty`, `LogprobsTensors`, `unsqueeze`, `compute_token_logprobs`, `max`, `new_zeros`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `empty`, `LogprobsTensors`, `unsqueeze`, `compute_token_logprobs`, `max`, `new_zeros`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_fill_logprob_token_ids_kernel` function / `_fill_logprob_token_ids_kernel` 函数
```python
@triton.jit
def _fill_logprob_token_ids_kernel(
    # [batch_size, 1 + num_cols]
    out_token_ids_ptr,
    out_token_ids_stride,
    # [batch_size, 1 + num_cols]
    out_valid_mask_ptr,
    out_valid_mask_stride,
    sampled_token_ids_ptr,  # [batch_size]
    topk_indices_ptr,  # [batch_size, NUM_TOPK] (unused when NUM_TOPK == 0)
    topk_indices_stride,
    expanded_idx_mapping_ptr,  # [batch_size] -> req_state_idx
    num_per_req_token_ids_ptr,  # [max_num_reqs]
    per_req_token_ids_ptr,  # [max_num_reqs, MAX_LOGPROB_TOKEN_IDS]
    per_req_token_ids_stride,
    NUM_TOPK: tl.constexpr,
    PADDED_COLS: tl.constexpr,
):
    batch_idx = tl.program_id(0)

    # Column 0: always the sampled token, always valid.
    sampled = tl.load(sampled_token_ids_ptr + batch_idx)
    tl.store(out_token_ids_ptr + batch_idx * out_token_ids_stride, sampled)
    tl.store(out_valid_mask_ptr + batch_idx * out_valid_mask_stride, 1)

    req_state_idx = tl.load(expanded_idx_mapping_ptr + batch_idx)
    num_custom = tl.load(num_per_req_token_ids_ptr + req_state_idx)

    col = tl.arange(0, PADDED_COLS)
    tid_base = out_token_ids_ptr + batch_idx * out_token_ids_stride + 1
    mask_base = out_valid_mask_ptr + batch_idx * out_valid_mask_stride + 1

    if num_custom > 0:
        # Override topk with per-request custom tokens.
        src = per_req_token_ids_ptr + req_state_idx * per_req_token_ids_stride
        valid = col < num_custom
    else:
        # Fill with topk indices (no-op when NUM_TOPK == 0).
        src = topk_indices_ptr + batch_idx * topk_indices_stride
        valid = col < NUM_TOPK

    tokens = tl.load(src + col, mask=valid, other=0).to(tl.int64)
    tl.store(tid_base + col, tokens, mask=valid)
    tl.store(mask_base + col, tl.full([PADDED_COLS], 1, tl.int1), mask=valid)
```
**EN:** This function implements `_fill_logprob_token_ids_kernel` within the module. Key calls include `program_id`, `load`, `store`, `arange`, `to`, `full`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_fill_logprob_token_ids_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `store`, `arange`, `to`, `full`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LogprobTokenIdsState` class / `LogprobTokenIdsState` 类
```python
class LogprobTokenIdsState:
    """Per-request override of which token ids' logprobs to return.

    See `SamplingParams.logprob_token_ids`.
    """
```
**EN:** Introduces the `LogprobTokenIdsState` class. Core methods include `__init__`, `add_request`, `apply_staged_writes`, `max_num_token_ids`. Docstring signal: Per-request override of which token ids' logprobs to return.
**CN:** 这里定义 `LogprobTokenIdsState` 类。核心方法包括 `__init__`, `add_request`, `apply_staged_writes`, `max_num_token_ids`。

### `LogprobTokenIdsState.__init__` method / `LogprobTokenIdsState.__init__` 方法
```python
    def __init__(self, max_num_reqs: int, device: torch.device):
        self.max_num_reqs = max_num_reqs
        self.num_token_ids = UvaBackedTensor(max_num_reqs, dtype=torch.int32)
        self.token_ids = StagedWriteTensor(
            (max_num_reqs, MAX_LOGPROB_TOKEN_IDS),
            dtype=torch.int32,
            device=device,
        )
```
**EN:** This method initializes the object state within `LogprobTokenIdsState`. Key calls include `UvaBackedTensor`, `StagedWriteTensor`. It touches state such as `max_num_reqs`, `num_token_ids`, `token_ids`.
**CN:** 该方法会初始化对象状态，其作用域位于`LogprobTokenIdsState`。 关键调用包括 `UvaBackedTensor`, `StagedWriteTensor`。 它会读写 `max_num_reqs`, `num_token_ids`, `token_ids` 等状态。

### `LogprobTokenIdsState.add_request` method / `LogprobTokenIdsState.add_request` 方法
```python
    def add_request(self, req_idx: int, sampling_params: SamplingParams) -> None:
        token_ids = sampling_params.logprob_token_ids
        if not token_ids:
            self.num_token_ids.np[req_idx] = 0
            return
        n = len(token_ids)
        if n > MAX_LOGPROB_TOKEN_IDS:
            raise ValueError(
                f"Too many logprob_token_ids: {n}. The max is {MAX_LOGPROB_TOKEN_IDS}."
            )
        self.num_token_ids.np[req_idx] = n
        self.token_ids.stage_write(req_idx, 0, token_ids)
```
**EN:** This method implements `add_request` within `LogprobTokenIdsState`. Key calls include `len`, `stage_write`, `ValueError`. It touches state such as `num_token_ids`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`LogprobTokenIdsState`。 关键调用包括 `len`, `stage_write`, `ValueError`。 它会读写 `num_token_ids` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LogprobTokenIdsState.apply_staged_writes` method / `LogprobTokenIdsState.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        self.num_token_ids.copy_to_uva()
        self.token_ids.apply_write()
```
**EN:** This method implements `apply_staged_writes` within `LogprobTokenIdsState`. Key calls include `copy_to_uva`, `apply_write`.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`LogprobTokenIdsState`。 关键调用包括 `copy_to_uva`, `apply_write`。

### `LogprobTokenIdsState.max_num_token_ids` method / `LogprobTokenIdsState.max_num_token_ids` 方法
```python
    def max_num_token_ids(self, idx_mapping_np: np.ndarray) -> int:
        return int(self.num_token_ids.np[idx_mapping_np].max(initial=0))
```
**EN:** This method implements `max_num_token_ids` within `LogprobTokenIdsState`. Key calls include `int`, `max`.
**CN:** 该方法会实现 `max_num_token_ids`，其作用域位于`LogprobTokenIdsState`。 关键调用包括 `int`, `max`。

## Key Concepts / 关键概念
- `_topk_log_softmax_kernel`: top-level helper or orchestration entry point. / `_topk_log_softmax_kernel`：顶层辅助函数或编排入口。
- `_ranks_kernel`: top-level helper or orchestration entry point. / `_ranks_kernel`：顶层辅助函数或编排入口。
- `compute_token_logprobs`: top-level helper or orchestration entry point. / `compute_token_logprobs`：顶层辅助函数或编排入口。
- `compute_topk_logprobs`: top-level helper or orchestration entry point. / `compute_topk_logprobs`：顶层辅助函数或编排入口。
- `_fill_logprob_token_ids_kernel`: top-level helper or orchestration entry point. / `_fill_logprob_token_ids_kernel`：顶层辅助函数或编排入口。
- `LogprobTokenIdsState`: central class or interface in this module. / `LogprobTokenIdsState`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.worker.gpu.buffer_utils`
