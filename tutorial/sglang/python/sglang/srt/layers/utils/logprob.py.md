# logprob.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/utils/logprob.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime layer support for the SGLang SRT runtime. It exposes symbols such as `LogprobStage`, `InputLogprobsResult`, `compute_temp_top_p_normalized_logprobs`, and `get_top_logprobs_raw` and connects them to backend-specific paths such as `NPU` and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了运行时层支持。它提供了 `LogprobStage`、`InputLogprobsResult`、`compute_temp_top_p_normalized_logprobs` 以及 `get_top_logprobs_raw` 等符号，并把这些符号连接到 `NPU` 和 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import dataclasses
from enum import Enum, auto
from typing import TYPE_CHECKING, List, Optional, Union

import torch

from sglang.srt.environ import envs

if TYPE_CHECKING:
    from sglang.srt.layers.logits_processor import LogitsMetadata, LogitsProcessorOutput
    from sglang.srt.managers.schedule_batch import ScheduleBatch
    from sglang.srt.speculative.eagle_info import EagleVerifyOutput
    from sglang.srt.speculative.ngram_info import NgramVerifyInput
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `dataclasses`, `enum.Enum`, `enum.auto`, `typing.TYPE_CHECKING`, and `typing.List`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`dataclasses`、`enum.Enum`、`enum.auto`、`typing.TYPE_CHECKING` 以及 `typing.List`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 18-22: Class `LogprobStage` declaration and shared state
```python
class LogprobStage(Enum):
    PREFILL = auto()
    DECODE = auto()
```
**EN:** This block introduces class `LogprobStage` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `LogprobStage`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 23-31: Class `InputLogprobsResult` declaration and shared state
```python
@dataclasses.dataclass
class InputLogprobsResult:
    input_token_logprobs: torch.Tensor
    input_top_logprobs_val: Optional[List] = None
    input_top_logprobs_idx: Optional[List] = None
    input_token_ids_logprobs_val: Optional[List] = None
    input_token_ids_logprobs_idx: Optional[List] = None
```
**EN:** This block introduces class `InputLogprobsResult` and the state shared by its methods.
**CN:** 该代码块引入类 `InputLogprobsResult`，并定义其方法共享的状态。

### Lines 32-65: `compute_temp_top_p_normalized_logprobs` step for temp top p normalized logprobs
```python
def compute_temp_top_p_normalized_logprobs(
    last_logits: torch.Tensor,
    logits_metadata: LogitsMetadata,
    top_p: Optional[torch.Tensor] = None,
    temperature: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """
    compute logprobs for the output token from the given logits.

    Returns:
        torch.Tensor: logprobs from logits
    """
    if top_p is None:
        top_p = logits_metadata.top_p
    if temperature is None:
        temperature = logits_metadata.temperature

    # Scale logits if temperature scaling is enabled
    if logits_metadata.temp_scaled_logprobs:
        last_logits = last_logits / temperature

    # Normalize logprobs if top_p normalization is enabled
    # NOTE: only normalize logprobs when top_p is set and not equal to 1.0
    if logits_metadata.top_p_normalized_logprobs and (top_p != 1.0).any():
        from sglang.srt.layers.sampler import top_p_normalize_probs_torch

        probs = torch.softmax(last_logits, dim=-1)
        del last_logits
        probs = top_p_normalize_probs_torch(probs, top_p)
        return torch.log(probs)
    else:
        return torch.nn.functional.log_softmax(last_logits, dim=-1)
```
**EN:** This block defines `compute_temp_top_p_normalized_logprobs` and contains the main logic for this step. It mainly invokes `any`, `torch.softmax`, `top_p_normalize_probs_torch`, `torch.log`, and `torch.nn.functional.log_softmax`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `top_p`, `temperature`, `last_logits`, and `probs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `compute_temp_top_p_normalized_logprobs`，并承载这一阶段的核心逻辑。 它主要调用 `any`、`torch.softmax`、`top_p_normalize_probs_torch`、`torch.log` 以及 `torch.nn.functional.log_softmax`，说明该流程会编排底层辅助函数或计算内核。 像 `top_p`、`temperature`、`last_logits` 以及 `probs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 66-100: `get_top_logprobs_raw` getter for top logprobs raw
```python
def get_top_logprobs_raw(
    logprobs: torch.Tensor,
    top_logprobs_nums: List[int],
    stage: LogprobStage,
    extend_logprob_pruned_lens_cpu: Optional[List[int]] = None,
    no_copy_to_cpu: bool = False,
):
    max_k = max(top_logprobs_nums)
    values, indices = logprobs.topk(max_k, dim=-1)
    if not no_copy_to_cpu:
        values = values.tolist()
        indices = indices.tolist()

    top_logprobs_val = []
    top_logprobs_idx = []

    if stage == LogprobStage.DECODE:
        for i, k in enumerate(top_logprobs_nums):
            top_logprobs_val.append(values[i][:k])
            top_logprobs_idx.append(indices[i][:k])
    else:
        pt = 0
        for k, pruned_len in zip(top_logprobs_nums, extend_logprob_pruned_lens_cpu):
            if pruned_len <= 0:
                top_logprobs_val.append([])
                top_logprobs_idx.append([])
                continue

            top_logprobs_val.append([values[pt + j][:k] for j in range(pruned_len)])
            top_logprobs_idx.append([indices[pt + j][:k] for j in range(pruned_len)])
            pt += pruned_len

    return top_logprobs_val, top_logprobs_idx
```
**EN:** This block defines `get_top_logprobs_raw` and contains the main logic for this step. It mainly invokes `max`, `logprobs.topk`, `values.tolist`, `indices.tolist`, and `enumerate`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `max_k`, `values`, `indices`, `top_logprobs_val`, and `top_logprobs_idx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_top_logprobs_raw`，并承载这一阶段的核心逻辑。 它主要调用 `max`、`logprobs.topk`、`values.tolist`、`indices.tolist` 以及 `enumerate`，说明该流程会编排底层辅助函数或计算内核。 像 `max_k`、`values`、`indices`、`top_logprobs_val` 以及 `top_logprobs_idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 101-111: `get_top_logprobs_prefill` getter for top logprobs prefill
```python
def get_top_logprobs_prefill(
    all_logprobs: torch.Tensor, logits_metadata: LogitsMetadata
):
    return get_top_logprobs_raw(
        all_logprobs,
        logits_metadata.top_logprobs_nums,
        stage=LogprobStage.PREFILL,
        extend_logprob_pruned_lens_cpu=logits_metadata.extend_logprob_pruned_lens_cpu,
    )
```
**EN:** This block defines `get_top_logprobs_prefill` and contains the main logic for this step. It mainly invokes `get_top_logprobs_raw`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_top_logprobs_prefill`，并承载这一阶段的核心逻辑。 它主要调用 `get_top_logprobs_raw`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 112-124: `get_top_logprobs` getter for top logprobs
```python
def get_top_logprobs(
    logprobs: torch.Tensor,
    top_logprobs_nums: List[int],
    no_copy_to_cpu: bool = False,
):
    return get_top_logprobs_raw(
        logprobs,
        top_logprobs_nums,
        stage=LogprobStage.DECODE,
        no_copy_to_cpu=no_copy_to_cpu,
    )
```
**EN:** This block defines `get_top_logprobs` and contains the main logic for this step. It mainly invokes `get_top_logprobs_raw`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_top_logprobs`，并承载这一阶段的核心逻辑。 它主要调用 `get_top_logprobs_raw`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 125-163: `get_token_ids_logprobs_raw` getter for token ids logprobs raw
```python
def get_token_ids_logprobs_raw(
    logprobs: torch.Tensor,
    token_ids_logprobs_list: List[Optional[List[int]]],
    stage: LogprobStage,
    extend_logprob_pruned_lens_cpu: Optional[List[int]] = None,
    no_copy_to_cpu: bool = False,
):
    vals, idxs = [], []
    if stage == LogprobStage.DECODE:
        for i, token_ids in enumerate(token_ids_logprobs_list):
            if token_ids is None:
                vals.append([])
                idxs.append([])
            else:
                token_ids_tensor = torch.tensor(token_ids, dtype=torch.long).to(
                    logprobs.device, non_blocking=True
                )
                row = logprobs[i, token_ids_tensor]
                vals.append(row if no_copy_to_cpu else row.tolist())
                idxs.append(token_ids)
    else:  # prefill
        pt = 0
        for i, (token_ids, pruned_len) in enumerate(
            zip(token_ids_logprobs_list, extend_logprob_pruned_lens_cpu)
        ):
            if pruned_len <= 0:
                vals.append([])
                idxs.append([])
                continue
            token_ids_tensor = torch.tensor(token_ids, dtype=torch.long).to(
                logprobs.device, non_blocking=True
            )
            pos_logprobs = logprobs[pt : pt + pruned_len, token_ids_tensor]
            vals.append(pos_logprobs if no_copy_to_cpu else pos_logprobs.tolist())
            idxs.append([token_ids for _ in range(pruned_len)])
            pt += pruned_len
    return vals, idxs
```
**EN:** This block defines `get_token_ids_logprobs_raw` and contains the main logic for this step. It mainly invokes `enumerate`, `zip`, `torch.tensor.to`, `vals.append`, and `idxs.append`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `vals`, `idxs`, `pt`, `token_ids_tensor`, and `pos_logprobs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_token_ids_logprobs_raw`，并承载这一阶段的核心逻辑。 它主要调用 `enumerate`、`zip`、`torch.tensor.to`、`vals.append` 以及 `idxs.append`，说明该流程会编排底层辅助函数或计算内核。 像 `vals`、`idxs`、`pt`、`token_ids_tensor` 以及 `pos_logprobs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 164-175: `get_token_ids_logprobs_prefill` getter for token ids logprobs prefill
```python
def get_token_ids_logprobs_prefill(
    all_logprobs, logits_metadata: LogitsMetadata, no_copy_to_cpu=False
):
    return get_token_ids_logprobs_raw(
        all_logprobs,
        logits_metadata.token_ids_logprobs,
        stage=LogprobStage.PREFILL,
        extend_logprob_pruned_lens_cpu=logits_metadata.extend_logprob_pruned_lens_cpu,
        no_copy_to_cpu=no_copy_to_cpu,
    )
```
**EN:** This block defines `get_token_ids_logprobs_prefill` and contains the main logic for this step. It mainly invokes `get_token_ids_logprobs_raw`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_token_ids_logprobs_prefill`，并承载这一阶段的核心逻辑。 它主要调用 `get_token_ids_logprobs_raw`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 176-184: `get_token_ids_logprobs` getter for token ids logprobs
```python
def get_token_ids_logprobs(logprobs, token_ids_logprobs, no_copy_to_cpu=False):
    return get_token_ids_logprobs_raw(
        logprobs,
        token_ids_logprobs,
        stage=LogprobStage.DECODE,
        no_copy_to_cpu=no_copy_to_cpu,
    )
```
**EN:** This block defines `get_token_ids_logprobs` and contains the main logic for this step. It mainly invokes `get_token_ids_logprobs_raw`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_token_ids_logprobs`，并承载这一阶段的核心逻辑。 它主要调用 `get_token_ids_logprobs_raw`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 185-258: `get_top_logprobs_chunk` getter for top logprobs chunk
```python
def get_top_logprobs_chunk(
    logprobs: torch.Tensor,
    logits_metadata: LogitsMetadata,
    top_k_nums: List[int],
    pruned_lens: List[int],
    input_top_logprobs_val: List,
    input_top_logprobs_idx: List,
    split_pruned_len: int,
) -> int:
    """Get top-k logprobs for each sequence in the chunk.

    Args:
        logprobs: Log probabilities tensor of shape [seq_len, vocab_size]
        logits_metadata: Metadata containing top-k and pruned length info
        top_k_nums: List of top-k numbers for each sequence
        pruned_lens: List of pruned lengths for each sequence
        input_top_logprobs_val: List to store top-k logprob values
        input_top_logprobs_idx: List to store top-k token indices
        split_pruned_len: Length of pruned tokens from previous chunk

    Returns:
        int: Number of remaining tokens to process in next chunk
    """
    # No sequences in the chunk
    if logprobs.shape[0] == 0:
        return 0

    max_k = max(logits_metadata.top_logprobs_nums)
    ret = logprobs.topk(max_k, dim=1)
    values = ret.values.tolist()
    indices = ret.indices.tolist()

    pt = 0
    next_split_pruned_len = 0
    for n, (k, pruned_len) in enumerate(zip(top_k_nums, pruned_lens)):
        if n == 0:
            # For the first sequence, adjust the pruned length
            pruned_len -= split_pruned_len
        else:
            # After the first sequence, no split in the middle
            split_pruned_len = 0

        if pruned_len <= 0:
            # if pruned length is less than or equal to 0,
            # there is no top-k logprobs to process
            input_top_logprobs_val.append([])
            input_top_logprobs_idx.append([])
            continue

        # Get the top-k logprobs
        val = []
        idx = []
        for j in range(pruned_len):
            # Handle remaining tokens in next chunk if any
            if pt + j >= len(values):
                next_split_pruned_len = split_pruned_len + j
                break
            # Append the top-k logprobs
            val.append(values[pt + j][:k])
            idx.append(indices[pt + j][:k])

        # Append or extend based on whether the sequence was split across chunks
        if len(val) > 0:
            if split_pruned_len > 0:
                input_top_logprobs_val[-1].extend(val)
                input_top_logprobs_idx[-1].extend(idx)
            else:
                input_top_logprobs_val.append(val)
                input_top_logprobs_idx.append(idx)

        pt += pruned_len
    return next_split_pruned_len
```
**EN:** This block defines `get_top_logprobs_chunk` and contains the main logic for this step. It mainly invokes `max`, `logprobs.topk`, `ret.values.tolist`, `ret.indices.tolist`, and `enumerate`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `max_k`, `ret`, `values`, `indices`, and `pt` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_top_logprobs_chunk`，并承载这一阶段的核心逻辑。 它主要调用 `max`、`logprobs.topk`、`ret.values.tolist`、`ret.indices.tolist` 以及 `enumerate`，说明该流程会编排底层辅助函数或计算内核。 像 `max_k`、`ret`、`values`、`indices` 以及 `pt` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 259-331: `get_token_ids_logprobs_chunk` getter for token ids logprobs chunk
```python
def get_token_ids_logprobs_chunk(
    logprobs: torch.Tensor,
    token_ids_logprobs: List[int],
    pruned_lens: List[int],
    input_token_ids_logprobs_val: List,
    input_token_ids_logprobs_idx: List,
    split_pruned_len: int = 0,
):
    """Get token_ids logprobs for each sequence in the chunk.

    Args:
        logprobs: Log probabilities tensor of shape [seq_len, vocab_size]
        logits_metadata: Metadata containing token IDs and pruned length info
        token_ids_logprobs: List of token IDs for each sequence
        pruned_lens: List of pruned lengths for each sequence
        input_token_ids_logprobs_val: List to store token logprob values
        input_token_ids_logprobs_idx: List to store token indices
        split_pruned_len: Length of pruned tokens from previous chunk

    Returns:
        int: Number of remaining tokens to process in next chunk
    """

    # No sequences in the chunk
    if logprobs.shape[0] == 0:
        return 0

    pt = 0
    next_split_pruned_len = 0
    for n, (token_ids, pruned_len) in enumerate(
        zip(
            token_ids_logprobs,
            pruned_lens,
        )
    ):
        # Adjust pruned length for first sequence
        if n == 0:
            pruned_len -= split_pruned_len
        else:
            split_pruned_len = 0

        if pruned_len <= 0:
            # if pruned length is less than or equal to 0,
            # there is no token ids logprobs to process
            input_token_ids_logprobs_val.append([])
            input_token_ids_logprobs_idx.append([])
            continue

        # Get the token ids logprobs
        val = []
        idx = []
        for j in range(pruned_len):
            # Handle remaining tokens in next chunk if any
            if pt + j >= logprobs.shape[0]:
                next_split_pruned_len = split_pruned_len + j
                break
            if token_ids is not None:
                val.append(logprobs[pt + j, token_ids].tolist())
                idx.append(token_ids)

        # Append or extend based on whether the sequence was split across chunks
        if len(val) > 0:
            if split_pruned_len > 0:
                input_token_ids_logprobs_val[-1].extend(val)
                input_token_ids_logprobs_idx[-1].extend(idx)
            else:
                input_token_ids_logprobs_val.append(val)
                input_token_ids_logprobs_idx.append(idx)

        pt += pruned_len
    return next_split_pruned_len
```
**EN:** This block defines `get_token_ids_logprobs_chunk` and contains the main logic for this step. It mainly invokes `enumerate`, `zip`, `range`, `input_token_ids_logprobs_val.append`, and `input_token_ids_logprobs_idx.append`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pt`, `next_split_pruned_len`, `val`, `idx`, and `pruned_len` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_token_ids_logprobs_chunk`，并承载这一阶段的核心逻辑。 它主要调用 `enumerate`、`zip`、`range`、`input_token_ids_logprobs_val.append` 以及 `input_token_ids_logprobs_idx.append`，说明该流程会编排底层辅助函数或计算内核。 像 `pt`、`next_split_pruned_len`、`val`、`idx` 以及 `pruned_len` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 332-431: Function `add_output_logprobs_for_spec_v1` and its core logic
```python
def add_output_logprobs_for_spec_v1(
    batch: ScheduleBatch,
    res: Union[EagleVerifyOutput, NgramVerifyInput],
    logits_output: Optional[LogitsProcessorOutput] = None,
):
    # Extract args
    if logits_output is None:
        logits_output = res.logits_output

    if hasattr(res, "num_correct_drafts_per_req_cpu"):
        num_correct_drafts_per_req_cpu = res.num_correct_drafts_per_req_cpu
    else:
        # FIXME: Get a NgramVerifyOutput class and use that instead of this hack.
        num_correct_drafts_per_req_cpu = res.num_correct_drafts.tolist()

    top_logprobs_nums = batch.top_logprobs_nums
    token_ids_logprobs = batch.token_ids_logprobs
    accept_indices = res.accept_indices
    assert len(accept_indices) == len(logits_output.next_token_logits)

    temperatures = batch.sampling_info.temperatures
    num_draft_tokens = batch.spec_info.draft_token_num
    # acceptance indices are the indices in a "flattened" batch.
    # dividing it to num_draft_tokens will yield the actual batch index.
    temperatures = temperatures[accept_indices // num_draft_tokens]
    if envs.SGLANG_RETURN_ORIGINAL_LOGPROB.get():
        logprobs = torch.nn.functional.log_softmax(
            logits_output.next_token_logits, dim=-1
        )
    else:
        logprobs = torch.nn.functional.log_softmax(
            logits_output.next_token_logits / temperatures, dim=-1
        )
    batch_next_token_ids = res.accept_tokens
    num_tokens_per_req = [accept + 1 for accept in num_correct_drafts_per_req_cpu]

    # We should repeat top_logprobs_nums to match num_tokens_per_req.
    top_logprobs_nums_repeat_interleaved = [
        num
        for num, num_tokens in zip(top_logprobs_nums, num_tokens_per_req)
        for _ in range(num_tokens)
    ]

    token_ids_logprobs_repeat_interleaved = [
        token_ids
        for token_ids, num_tokens in zip(token_ids_logprobs, num_tokens_per_req)
        for _ in range(num_tokens)
    ]

    # Extract logprobs
    should_top_logprobs = any(x > 0 for x in top_logprobs_nums)
    should_token_ids_logprobs = any(x is not None for x in token_ids_logprobs)
    if should_top_logprobs:
        (
            logits_output.next_token_top_logprobs_val,
            logits_output.next_token_top_logprobs_idx,
        ) = get_top_logprobs(
            logprobs,
            top_logprobs_nums_repeat_interleaved,
        )

    if should_token_ids_logprobs:
        (
            logits_output.next_token_token_ids_logprobs_val,
            logits_output.next_token_token_ids_logprobs_idx,
        ) = get_token_ids_logprobs(
            logprobs,
            token_ids_logprobs_repeat_interleaved,
        )

    logits_output.next_token_logprobs = logprobs[
        torch.arange(len(batch_next_token_ids), device=batch.sampling_info.device),
        batch_next_token_ids,
    ]

    # Add output logprobs to the request
    pt = 0
    next_token_logprobs = logits_output.next_token_logprobs.tolist()
    accept_tokens_list = batch_next_token_ids.tolist()
    token_top_logprobs_val = logits_output.next_token_top_logprobs_val
    token_top_logprobs_idx = logits_output.next_token_top_logprobs_idx
    token_ids_logprobs_val = logits_output.next_token_token_ids_logprobs_val
    token_ids_logprobs_idx = logits_output.next_token_token_ids_logprobs_idx
    for req, num_tokens in zip(batch.reqs, num_tokens_per_req, strict=True):
        for _ in range(num_tokens):
            if req.return_logprob:
                req.output_token_logprobs_val.append(next_token_logprobs[pt])
                req.output_token_logprobs_idx.append(accept_tokens_list[pt])
                if req.top_logprobs_num > 0:
                    assert (
                        should_top_logprobs
                    ), "Inconsistent state: should_top_logprobs is False"
                    req.output_top_logprobs_val.append(token_top_logprobs_val[pt])
                    req.output_top_logprobs_idx.append(token_top_logprobs_idx[pt])
                if req.token_ids_logprob is not None and should_token_ids_logprobs:
                    req.output_token_ids_logprobs_val.append(token_ids_logprobs_val[pt])
                    req.output_token_ids_logprobs_idx.append(token_ids_logprobs_idx[pt])
            pt += 1
```
**EN:** This block defines `add_output_logprobs_for_spec_v1` and contains the main logic for this step. It mainly invokes `hasattr`, `envs.SGLANG_RETURN_ORIGINAL_LOGPROB.get`, `any`, `logits_output.next_token_logprobs.tolist`, and `batch_next_token_ids.tolist`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `top_logprobs_nums`, `token_ids_logprobs`, `accept_indices`, `temperatures`, and `num_draft_tokens` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `add_output_logprobs_for_spec_v1`，并承载这一阶段的核心逻辑。 它主要调用 `hasattr`、`envs.SGLANG_RETURN_ORIGINAL_LOGPROB.get`、`any`、`logits_output.next_token_logprobs.tolist` 以及 `batch_next_token_ids.tolist`，说明该流程会编排底层辅助函数或计算内核。 像 `top_logprobs_nums`、`token_ids_logprobs`、`accept_indices`、`temperatures` 以及 `num_draft_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 432-494: `compute_spec_v2_logprobs` step for spec v 2 logprobs
```python
def compute_spec_v2_logprobs(
    batch,
    logits_output,
    predict: torch.Tensor,
    accept_index: torch.Tensor,
    speculative_num_steps: int,
):
    """Compute logprobs for accepted tokens after spec v2 verify sampling.

    Gathers logits at accepted positions, applies log_softmax (temperature-scaled
    if not greedy), and populates logits_output.next_token_logprobs (plus optional
    top-k / token-ids logprobs) so they flow through copy_to_cpu().
    """
    bs = len(batch.seq_lens)
    max_accept = speculative_num_steps + 1
    device = predict.device

    flat_accept_idx = accept_index.long().reshape(-1)
    gathered_logits = logits_output.next_token_logits[flat_accept_idx]

    if batch.sampling_info.is_all_greedy or envs.SGLANG_RETURN_ORIGINAL_LOGPROB.get():
        gathered_logprobs = torch.nn.functional.log_softmax(gathered_logits, dim=-1)
    else:
        temperatures = torch.repeat_interleave(
            batch.sampling_info.temperatures,
            max_accept,
            dim=0,
        )
        gathered_logprobs = torch.nn.functional.log_softmax(
            gathered_logits / temperatures, dim=-1
        )
    gathered_logprobs.clamp_(min=torch.finfo(gathered_logprobs.dtype).min)

    accepted_token_ids = predict[flat_accept_idx]
    token_logprobs = gathered_logprobs[
        torch.arange(bs * max_accept, device=device),
        accepted_token_ids.long(),
    ]
    logits_output.next_token_logprobs = token_logprobs.reshape(bs, max_accept)

    if batch.top_logprobs_nums and any(x > 0 for x in batch.top_logprobs_nums):
        top_logprobs_nums_expanded = [
            num for num in batch.top_logprobs_nums for _ in range(max_accept)
        ]
        (
            logits_output.next_token_top_logprobs_val,
            logits_output.next_token_top_logprobs_idx,
        ) = get_top_logprobs(
            gathered_logprobs, top_logprobs_nums_expanded, no_copy_to_cpu=True
        )

    if batch.token_ids_logprobs and any(
        x is not None for x in batch.token_ids_logprobs
    ):
        token_ids_logprobs_expanded = [
            ids for ids in batch.token_ids_logprobs for _ in range(max_accept)
        ]
        (
            logits_output.next_token_token_ids_logprobs_val,
            logits_output.next_token_token_ids_logprobs_idx,
        ) = get_token_ids_logprobs(
            gathered_logprobs, token_ids_logprobs_expanded, no_copy_to_cpu=True
        )
```
**EN:** This block defines `compute_spec_v2_logprobs` and contains the main logic for this step. It mainly invokes `len`, `accept_index.long.reshape`, `gathered_logprobs.clamp_`, `token_logprobs.reshape`, and `envs.SGLANG_RETURN_ORIGINAL_LOGPROB.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `max_accept`, `device`, `flat_accept_idx`, and `gathered_logits` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `compute_spec_v2_logprobs`，并承载这一阶段的核心逻辑。 它主要调用 `len`、`accept_index.long.reshape`、`gathered_logprobs.clamp_`、`token_logprobs.reshape` 以及 `envs.SGLANG_RETURN_ORIGINAL_LOGPROB.get`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`max_accept`、`device`、`flat_accept_idx` 以及 `gathered_logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `LogprobStage`, `InputLogprobsResult`, `compute_temp_top_p_normalized_logprobs`, `get_top_logprobs_raw`, and `get_top_logprobs_prefill`. / **主要符号**：核心入口包括 `LogprobStage`、`InputLogprobsResult`、`compute_temp_top_p_normalized_logprobs`、`get_top_logprobs_raw` 以及 `get_top_logprobs_prefill`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Sampling policy**: Covers post-processing of logits and token selection strategies. / **采样策略**：涵盖 logits 后处理与 token 选择策略。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `dataclasses`, `enum.Enum`, `enum.auto`, `typing.TYPE_CHECKING`, `typing.List`, `typing.Optional`, and `typing.Union` / **标准库**：`__future__.annotations`、`dataclasses`、`enum.Enum`、`enum.auto`、`typing.TYPE_CHECKING`、`typing.List`、`typing.Optional` 以及 `typing.Union`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.environ.envs`, `sglang.srt.layers.logits_processor.LogitsMetadata`, `sglang.srt.layers.logits_processor.LogitsProcessorOutput`, `sglang.srt.managers.schedule_batch.ScheduleBatch`, `sglang.srt.speculative.eagle_info.EagleVerifyOutput`, `sglang.srt.speculative.ngram_info.NgramVerifyInput`, and `sglang.srt.layers.sampler.top_p_normalize_probs_torch` / **SGLang 内部模块**：`sglang.srt.environ.envs`、`sglang.srt.layers.logits_processor.LogitsMetadata`、`sglang.srt.layers.logits_processor.LogitsProcessorOutput`、`sglang.srt.managers.schedule_batch.ScheduleBatch`、`sglang.srt.speculative.eagle_info.EagleVerifyOutput`、`sglang.srt.speculative.ngram_info.NgramVerifyInput` 以及 `sglang.srt.layers.sampler.top_p_normalize_probs_torch`
