# forward_batch_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/forward_batch_info.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `forward_batch_info`. The module docstring frames it as: "Store information about a forward batch." / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `forward_batch_info` 的逻辑。 它对外提供的主要入口包括 `ForwardMode`, `CaptureHiddenMode`, `compute_local_num_token_non_padded`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup (part 1/2)
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""
Store information about a forward batch.

The following is the flow of data structures for a batch:

ScheduleBatch -> ForwardBatch

- ScheduleBatch is managed by `scheduler.py::Scheduler`.
  It contains high-level scheduling data. Most of the data is on the CPU.
- ForwardBatch is managed by `model_runner.py::ModelRunner`.
  It contains low-level tensor data. Most of the data consists of GPU tensors.
  It is constructed directly from a ScheduleBatch by `ForwardBatch.init_new`.
"""

from __future__ import annotations

from dataclasses import dataclass
from enum import IntEnum, auto
from functools import total_ordering
from typing import TYPE_CHECKING, Dict, List, Optional, Tuple, Union

import torch
import triton
import triton.language as tl

from sglang.srt.distributed.parallel_state import (
    get_moe_expert_parallel_world_size,
    get_tensor_model_parallel_world_size,
)
from sglang.srt.layers.dp_attention import (
    DpPaddingMode,
    get_attention_cp_size,
    get_attention_dp_rank,
    get_attention_tp_rank,
    get_attention_tp_size,
    set_dp_buffer_len,
    set_is_extend_in_batch,
)
from sglang.srt.layers.utils.cp_utils import ContextParallelMetadata
from sglang.srt.model_executor.forward_batch_deepseek_mha_mixin import (
    ForwardBatchDeepSeekMHAMixin,
)
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    is_cuda,
    is_hip,
    is_npu,
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 61-77: Module imports, constants, and setup (part 2/2)
```python
    support_triton,
)
from sglang.srt.utils.common import ceil_align

if TYPE_CHECKING:
    from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
    from sglang.srt.layers.logits_processor import LogitsProcessorOutput
    from sglang.srt.managers.hisparse_coordinator import HiSparseCoordinator
    from sglang.srt.managers.schedule_batch import MultimodalInputs, ScheduleBatch
    from sglang.srt.mem_cache.memory_pool import KVCache, ReqToTokenPool
    from sglang.srt.model_executor.model_runner import ModelRunner
    from sglang.srt.sampling.sampling_batch_info import SamplingBatchInfo
    from sglang.srt.speculative.spec_info import SpecInput, SpeculativeAlgorithm

_is_npu = is_npu()


```
**EN:** This range organizes module-level state and shared setup. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 78-105: Class ForwardMode
```python
class ForwardMode(IntEnum):
    # Extend a sequence. The KV cache of the beginning part of the sequence is already computed (e.g., system prompt).
    # It is also called "prefill" in common terminology.
    EXTEND = auto()
    # Decode one token.
    DECODE = auto()
    # Contains both EXTEND and DECODE when doing chunked prefill.
    MIXED = auto()
    # No sequence to forward. For data parallel attention, some workers will be IDLE if no sequence are allocated.
    IDLE = auto()

    # Used in speculative decoding: verify a batch in the target model.
    TARGET_VERIFY = auto()
    # Used in speculative decoding: extend a batch in the draft model.
    DRAFT_EXTEND = auto()

    DRAFT_EXTEND_V2 = auto()

    # Used in disaggregated decode worker
    # Represent a batch of requests having their KV cache ready to start decoding
    PREBUILT = auto()

    # Split Prefill for PD multiplexing
    SPLIT_PREFILL = auto()

    # Used in dLLM
    DLLM_EXTEND = auto()

```
**EN:** This range introduces `ForwardMode` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ForwardMode`，并定义其后续方法依赖的结构或元数据。

### Lines 106-107: Method ForwardMode.is_prefill
```python
    def is_prefill(self, include_draft_extend_v2: bool = False):
        return self.is_extend(include_draft_extend_v2=include_draft_extend_v2)
```
**EN:** This callable implements `ForwardMode.is_prefill`. It takes `include_draft_extend_v2` and mainly implements is prefill.
**CN:** 这一可调用对象实现了 `ForwardMode.is_prefill`。它接收 `include_draft_extend_v2`，主要用于实现 is prefill 相关逻辑。

### Lines 109-118: Method ForwardMode.is_extend
```python
    def is_extend(self, include_draft_extend_v2: bool = False):
        return (
            self == ForwardMode.EXTEND
            or self == ForwardMode.MIXED
            or self == ForwardMode.DRAFT_EXTEND
            or (include_draft_extend_v2 and self == ForwardMode.DRAFT_EXTEND_V2)
            or self == ForwardMode.TARGET_VERIFY
            or self == ForwardMode.SPLIT_PREFILL
            or self == ForwardMode.DLLM_EXTEND
        )
```
**EN:** This callable implements `ForwardMode.is_extend`. It takes `include_draft_extend_v2` and mainly implements is extend.
**CN:** 这一可调用对象实现了 `ForwardMode.is_extend`。它接收 `include_draft_extend_v2`，主要用于实现 is extend 相关逻辑。

### Lines 120-129: Method ForwardMode.is_context_parallel_extend
```python
    def is_context_parallel_extend(self, include_draft_extend_v2: bool = False):
        return (
            self == ForwardMode.EXTEND
            or self == ForwardMode.MIXED
            or (
                self == ForwardMode.DRAFT_EXTEND_V2
                if include_draft_extend_v2
                else False
            )
        )
```
**EN:** This callable implements `ForwardMode.is_context_parallel_extend`. It takes `include_draft_extend_v2` and mainly implements is context parallel extend.
**CN:** 这一可调用对象实现了 `ForwardMode.is_context_parallel_extend`。它接收 `include_draft_extend_v2`，主要用于实现 is context parallel extend 相关逻辑。

### Lines 131-132: Method ForwardMode.is_decode
```python
    def is_decode(self):
        return self == ForwardMode.DECODE
```
**EN:** This callable implements `ForwardMode.is_decode` and mainly implements is decode.
**CN:** 这一可调用对象实现了 `ForwardMode.is_decode`，主要用于实现 is decode 相关逻辑。

### Lines 134-135: Method ForwardMode.is_mixed
```python
    def is_mixed(self):
        return self == ForwardMode.MIXED
```
**EN:** This callable implements `ForwardMode.is_mixed` and mainly implements is mixed.
**CN:** 这一可调用对象实现了 `ForwardMode.is_mixed`，主要用于实现 is mixed 相关逻辑。

### Lines 137-138: Method ForwardMode.is_idle
```python
    def is_idle(self):
        return self == ForwardMode.IDLE
```
**EN:** This callable implements `ForwardMode.is_idle` and mainly implements is idle.
**CN:** 这一可调用对象实现了 `ForwardMode.is_idle`，主要用于实现 is idle 相关逻辑。

### Lines 140-141: Method ForwardMode.is_decode_or_idle
```python
    def is_decode_or_idle(self):
        return self == ForwardMode.DECODE or self == ForwardMode.IDLE
```
**EN:** This callable implements `ForwardMode.is_decode_or_idle` and mainly implements is decode or idle.
**CN:** 这一可调用对象实现了 `ForwardMode.is_decode_or_idle`，主要用于实现 is decode or idle 相关逻辑。

### Lines 143-144: Method ForwardMode.is_target_verify
```python
    def is_target_verify(self):
        return self == ForwardMode.TARGET_VERIFY
```
**EN:** This callable implements `ForwardMode.is_target_verify` and mainly validates inputs and invariants.
**CN:** 这一可调用对象实现了 `ForwardMode.is_target_verify`，主要用于校验输入与不变量。

### Lines 146-149: Method ForwardMode.is_draft_extend
```python
    def is_draft_extend(self, include_v2: bool = False):
        return self == ForwardMode.DRAFT_EXTEND or (
            include_v2 and self == ForwardMode.DRAFT_EXTEND_V2
        )
```
**EN:** This callable implements `ForwardMode.is_draft_extend`. It takes `include_v2` and mainly implements is draft extend.
**CN:** 这一可调用对象实现了 `ForwardMode.is_draft_extend`。它接收 `include_v2`，主要用于实现 is draft extend 相关逻辑。

### Lines 151-153: Method ForwardMode.is_draft_extend_v2
```python
    def is_draft_extend_v2(self):
        # For fixed shape logits output in eagle v2 worker
        return self == ForwardMode.DRAFT_EXTEND_V2
```
**EN:** This callable implements `ForwardMode.is_draft_extend_v2` and mainly implements is draft extend v2.
**CN:** 这一可调用对象实现了 `ForwardMode.is_draft_extend_v2`，主要用于实现 is draft extend v2 相关逻辑。

### Lines 155-162: Method ForwardMode.is_extend_or_draft_extend_or_mixed
```python
    def is_extend_or_draft_extend_or_mixed(self, include_draft_extend_v2: bool = False):
        return (
            self == ForwardMode.EXTEND
            or self == ForwardMode.DRAFT_EXTEND
            or self == ForwardMode.MIXED
            or self == ForwardMode.SPLIT_PREFILL
            or (include_draft_extend_v2 and self == ForwardMode.DRAFT_EXTEND_V2)
        )
```
**EN:** This callable implements `ForwardMode.is_extend_or_draft_extend_or_mixed`. It takes `include_draft_extend_v2` and mainly implements is extend or draft extend or mixed.
**CN:** 这一可调用对象实现了 `ForwardMode.is_extend_or_draft_extend_or_mixed`。它接收 `include_draft_extend_v2`，主要用于实现 is extend or draft extend or mixed 相关逻辑。

### Lines 164-170: Method ForwardMode.is_cuda_graph
```python
    def is_cuda_graph(self):
        return (
            self == ForwardMode.DECODE
            or self == ForwardMode.TARGET_VERIFY
            or self == ForwardMode.IDLE
            or self == ForwardMode.DLLM_EXTEND
        )
```
**EN:** This callable implements `ForwardMode.is_cuda_graph` and mainly implements is cuda graph. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `ForwardMode.is_cuda_graph`，主要用于实现 is cuda graph 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 172-173: Method ForwardMode.is_cpu_graph
```python
    def is_cpu_graph(self):
        return self == ForwardMode.DECODE
```
**EN:** This callable implements `ForwardMode.is_cpu_graph` and mainly implements is cpu graph. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `ForwardMode.is_cpu_graph`，主要用于实现 is cpu graph 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 175-176: Method ForwardMode.is_split_prefill
```python
    def is_split_prefill(self):
        return self == ForwardMode.SPLIT_PREFILL
```
**EN:** This callable implements `ForwardMode.is_split_prefill` and mainly implements is split prefill.
**CN:** 这一可调用对象实现了 `ForwardMode.is_split_prefill`，主要用于实现 is split prefill 相关逻辑。

### Lines 178-183: Method ForwardMode.is_extend_without_speculative
```python
    def is_extend_without_speculative(self):
        return (
            self.is_extend()
            and not self.is_target_verify()
            and not self.is_draft_extend()
        )
```
**EN:** This callable implements `ForwardMode.is_extend_without_speculative` and mainly implements is extend without speculative.
**CN:** 这一可调用对象实现了 `ForwardMode.is_extend_without_speculative`，主要用于实现 is extend without speculative 相关逻辑。

### Lines 185-186: Method ForwardMode.is_prebuilt
```python
    def is_prebuilt(self):
        return self == ForwardMode.PREBUILT
```
**EN:** This callable implements `ForwardMode.is_prebuilt` and mainly implements is prebuilt.
**CN:** 这一可调用对象实现了 `ForwardMode.is_prebuilt`，主要用于实现 is prebuilt 相关逻辑。

### Lines 188-189: Method ForwardMode.is_dllm_extend
```python
    def is_dllm_extend(self):
        return self == ForwardMode.DLLM_EXTEND
```
**EN:** This callable implements `ForwardMode.is_dllm_extend` and mainly implements is dllm extend.
**CN:** 这一可调用对象实现了 `ForwardMode.is_dllm_extend`，主要用于实现 is dllm extend 相关逻辑。

### Lines 192-200: Class CaptureHiddenMode
```python
@total_ordering
class CaptureHiddenMode(IntEnum):
    # Do not capture anything.
    NULL = 0
    # Capture a hidden state of the last token.
    LAST = 1
    # Capture hidden states of all tokens.
    FULL = 2

```
**EN:** This range introduces `CaptureHiddenMode` and defines the structure or metadata that its methods rely on. In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `CaptureHiddenMode`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 201-202: Method CaptureHiddenMode.need_capture
```python
    def need_capture(self):
        return self != CaptureHiddenMode.NULL
```
**EN:** This callable implements `CaptureHiddenMode.need_capture` and mainly implements need capture. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CaptureHiddenMode.need_capture`，主要用于实现 need capture 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 204-205: Method CaptureHiddenMode.is_full
```python
    def is_full(self):
        return self == CaptureHiddenMode.FULL
```
**EN:** This callable implements `CaptureHiddenMode.is_full` and mainly implements is full. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CaptureHiddenMode.is_full`，主要用于实现 is full 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 207-208: Method CaptureHiddenMode.is_last
```python
    def is_last(self):
        return self == CaptureHiddenMode.LAST
```
**EN:** This callable implements `CaptureHiddenMode.is_last` and mainly implements is last. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CaptureHiddenMode.is_last`，主要用于实现 is last 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 210-211: Method CaptureHiddenMode.__lt__
```python
    def __lt__(self, other):
        return self.value < other.value
```
**EN:** This callable implements `CaptureHiddenMode.__lt__`. It takes `other` and mainly implements lt.
**CN:** 这一可调用对象实现了 `CaptureHiddenMode.__lt__`。它接收 `other`，主要用于实现 lt 相关逻辑。

### Lines 214-231: Function compute_local_num_token_non_padded
```python
def compute_local_num_token_non_padded(
    global_num_token_non_padded: torch.Tensor,
    num_tokens_per_dp: int,
) -> torch.Tensor:
    """Compute local non-padded token count for this attention-TP rank.

    Converts a global count (across all TP ranks) to a local count for this rank.
    The "global" scope is within the current DP rank; DP is handled via num_tokens_per_dp.
    """
    attn_tp_rank = get_attention_tp_rank()
    attn_tp_size = get_attention_tp_size()
    tokens_per_rank = num_tokens_per_dp // attn_tp_size

    return torch.clamp(
        global_num_token_non_padded - tokens_per_rank * attn_tp_rank,
        0,
        tokens_per_rank,
    )
```
**EN:** This callable implements `compute_local_num_token_non_padded`. It takes `global_num_token_non_padded`, `num_tokens_per_dp` and mainly adds configuration entries or arguments. The docstring states: "Compute local non-padded token count for this attention-TP rank."
**CN:** 这一可调用对象实现了 `compute_local_num_token_non_padded`。它接收 `global_num_token_non_padded`, `num_tokens_per_dp`，主要用于添加配置项或参数。

### Lines 234-243: Class NgramEmbeddingInfo
```python
@dataclass
class NgramEmbeddingInfo:
    """Ngram embedding state for LongCat models."""

    token_table: torch.Tensor
    column_starts: torch.Tensor
    req_lens: torch.Tensor
    out_column_starts: torch.Tensor
    out_req_lens: torch.Tensor

```
**EN:** This range introduces `NgramEmbeddingInfo` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Ngram embedding state for LongCat models."
**CN:** 这一段引入 `NgramEmbeddingInfo`，并定义其后续方法依赖的结构或元数据。

### Lines 244-264: Method NgramEmbeddingInfo.create
```python
    @classmethod
    def create(
        cls,
        token_table: torch.Tensor,
        batch_size: int,
        device: torch.device,
        column_starts=None,
        req_lens=None,
    ) -> NgramEmbeddingInfo:
        info = cls(
            token_table=token_table,
            column_starts=torch.empty(batch_size, dtype=torch.int32, device=device),
            req_lens=torch.empty(batch_size, dtype=torch.int32, device=device),
            out_column_starts=torch.empty(batch_size, dtype=torch.int32, device=device),
            out_req_lens=torch.empty(batch_size, dtype=torch.int32, device=device),
        )
        if column_starts is not None:
            info.column_starts[:] = column_starts
        if req_lens is not None:
            info.req_lens[:] = req_lens
        return info
```
**EN:** This callable implements `NgramEmbeddingInfo.create`. It takes `cls`, `token_table`, `batch_size`, `device` and mainly constructs new objects or contexts.
**CN:** 这一可调用对象实现了 `NgramEmbeddingInfo.create`。它接收 `cls`, `token_table`, `batch_size`, `device`，主要用于构造新的对象或上下文。

### Lines 266-273: Method NgramEmbeddingInfo.slice
```python
    def slice(self, bs: int) -> NgramEmbeddingInfo:
        return NgramEmbeddingInfo(
            token_table=self.token_table,
            column_starts=self.column_starts[:bs],
            req_lens=self.req_lens[:bs],
            out_column_starts=self.out_column_starts[:bs],
            out_req_lens=self.out_req_lens[:bs],
        )
```
**EN:** This callable implements `NgramEmbeddingInfo.slice`. It takes `bs` and mainly implements slice.
**CN:** 这一可调用对象实现了 `NgramEmbeddingInfo.slice`。它接收 `bs`，主要用于实现 slice 相关逻辑。

### Lines 276-335: Class ForwardBatch (part 1/3)
```python
@dataclass
class ForwardBatch(ForwardBatchDeepSeekMHAMixin):
    """Store all inputs of a forward pass."""

    # The forward mode
    forward_mode: ForwardMode
    # The batch size
    batch_size: int
    # The input ids
    input_ids: torch.Tensor
    # The indices of requests in the req_to_token_pool
    req_pool_indices: torch.Tensor
    # The sequence length
    seq_lens: torch.Tensor
    # The indices of output tokens in the token_to_kv_pool
    out_cache_loc: torch.Tensor

    # The sum of all sequence lengths
    seq_lens_sum: int

    # The original sequence length without being chunked. Qwen-1M related.
    orig_seq_lens: Optional[torch.Tensor] = None

    # The indices of output tokens in the token_to_kv_pool_swa
    out_cache_loc_swa: Optional[torch.Tensor] = None
    # The indices to track mamba state with
    mamba_track_indices: Optional[torch.Tensor] = None  # shape: [b], int64
    # The mask to track mamba state if needed
    mamba_track_mask: Optional[torch.Tensor] = None  # shape: [b], bool
    # The seqlens to track mamba state if masked, prefill only.
    mamba_track_seqlens: Optional[torch.Tensor] = None  # shape: [b], int64

    # Optional seq_lens on cpu
    seq_lens_cpu: Optional[torch.Tensor] = None

    # For logprob
    return_logprob: bool = False
    top_logprobs_nums: Optional[List[int]] = None
    token_ids_logprobs: Optional[List[List[int]]] = None

    # For logits and logprobs post processing
    next_token_logits_buffer: torch.Tensor = None
    temp_scaled_logprobs: bool = False
    temperature: torch.Tensor = None
    top_p_normalized_logprobs: bool = False
    top_p: torch.Tensor = None

    # Position information
    positions: torch.Tensor = None

    # For extend
    extend_num_tokens: Optional[int] = None
    extend_seq_lens: Optional[torch.Tensor] = None
    extend_prefix_lens: Optional[torch.Tensor] = None
    extend_start_loc: Optional[torch.Tensor] = None
    extend_prefix_lens_cpu: Optional[List[int]] = None
    extend_seq_lens_cpu: Optional[List[int]] = None
    extend_logprob_start_lens_cpu: Optional[List[int]] = None
    extend_input_logprob_token_ids_gpu: Optional[torch.Tensor] = None

```
**EN:** This range introduces `ForwardBatch` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Store all inputs of a forward pass." This chunk is part 1 of 3 for the same logical block. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一段引入 `ForwardBatch`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 1/3 部分。 在这一范围内，它会调整采样行为与解码控制。

### Lines 336-395: Class ForwardBatch (part 2/3)
```python
    # For split prefill
    # intermediate values for split prefill
    hidden_states: torch.Tensor = None
    residual: torch.Tensor = None
    model_specific_states: Dict[str, any] = None
    # current split index of layer
    split_index: int = 0

    # For multimodal
    mm_inputs: Optional[List[MultimodalInputs]] = None

    # Encoder-decoder
    encoder_cached: Optional[List[bool]] = None
    encoder_lens: Optional[torch.Tensor] = None
    encoder_lens_cpu: Optional[List[int]] = None
    encoder_out_cache_loc: Optional[torch.Tensor] = None
    cross_attention_custom_mask: Optional[torch.Tensor] = None

    # For LoRA
    lora_ids: Optional[List[str]] = None

    # For input embeddings
    input_embeds: Optional[torch.Tensor] = None

    # For token embedding overrides (sparse replacement at specific positions)
    replace_embeds: Optional[torch.Tensor] = None
    replace_positions: Optional[torch.Tensor] = None

    # For cross-encoder model
    token_type_ids: Optional[torch.Tensor] = None

    # Sampling info
    sampling_info: SamplingBatchInfo = None

    # Attention backend
    req_to_token_pool: ReqToTokenPool = None
    token_to_kv_pool: KVCache = None
    attn_backend: AttentionBackend = None

    # For DP attention
    original_global_num_tokens_cpu: Optional[List[int]] = None
    global_num_tokens_cpu: Optional[List[int]] = None
    global_num_tokens_gpu: Optional[torch.Tensor] = None
    # Has to be None when cuda graph is captured.
    global_num_tokens_for_logprob_cpu: Optional[List[int]] = None
    global_num_tokens_for_logprob_gpu: Optional[torch.Tensor] = None
    # The padding mode for DP attention
    dp_padding_mode: Optional[DpPaddingMode] = None
    # for extend, local start pos and num tokens is different in logits processor
    # this will be computed in get_dp_local_info
    # this will be recomputed in LogitsMetadata.from_forward_batch
    dp_local_start_pos: Optional[torch.Tensor] = None  # cached info at runtime
    dp_local_num_tokens: Optional[torch.Tensor] = None  # cached info at runtime
    global_dp_buffer_len: Optional[int] = None
    is_extend_in_batch: bool = False
    # Mirrors ScheduleBatch.all_extend_in_batch; kept for downstream forks.
    all_extend_in_batch: bool = False
    can_run_dp_cuda_graph: bool = False
    global_forward_mode: Optional[ForwardMode] = None

```
**EN:** This range introduces `ForwardBatch` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Store all inputs of a forward pass." This chunk is part 2 of 3 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `ForwardBatch`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 2/3 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 396-441: Class ForwardBatch (part 3/3)
```python
    # Whether this batch is prefill-only (no token generation needed)
    is_prefill_only: bool = False

    # Pre-computed delimiter indices for multi-item scoring (CPU tensors, one per request)
    multi_item_delimiter_indices: Optional[List[torch.Tensor]] = None

    # Speculative decoding
    spec_info: Optional[SpecInput] = None
    spec_algorithm: SpeculativeAlgorithm = None
    mm_input_embeds: Optional[torch.Tensor] = None
    capture_hidden_mode: CaptureHiddenMode = None

    # For padding
    padded_static_len: int = -1  # -1 if not padded
    num_token_non_padded: Optional[torch.Tensor] = None  # scalar tensor
    num_token_non_padded_cpu: int = None

    # For Qwen2-VL
    mrope_positions: torch.Tensor = None

    # For two-batch overlap
    tbo_split_seq_index: Optional[int] = None
    tbo_parent_token_range: Optional[Tuple[int, int]] = None
    tbo_padded_len: Optional[int] = None
    tbo_children: Optional[List[ForwardBatch]] = None

    # For matryoshka embeddings
    dimensions: Optional[list[int]] = None

    attn_cp_metadata: Optional[ContextParallelMetadata] = None

    # For hidden states before normal
    return_hidden_states_before_norm: bool = False

    # Whether to return pooled hidden states (pre-head transformer output)
    return_pooled_hidden_states: bool = False

    # For hisparse
    hisparse_coordinator: Optional[HiSparseCoordinator] = None

    # For ngram embedding
    ngram_embedding_info: Optional[NgramEmbeddingInfo] = None

    # For dumper: request IDs for cross-step sequence tracking
    rids: Optional[List[str]] = None

```
**EN:** This range introduces `ForwardBatch` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Store all inputs of a forward pass." This chunk is part 3 of 3 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `ForwardBatch`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 3/3 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 442-501: Method ForwardBatch.init_new (part 1/4)
```python
    @classmethod
    def init_new(
        cls,
        batch: ScheduleBatch,
        model_runner: ModelRunner,
    ):
        # Consume one-shot per-forward overrides from SB; reset to defaults so
        # the next forward on the same SB starts clean. See SB field comment
        # for the contract.
        capture_hidden_mode = batch.capture_hidden_mode
        batch.capture_hidden_mode = None
        seq_lens_cpu_cache = batch.seq_lens_cpu_cache
        batch.seq_lens_cpu_cache = None
        return_hidden_states_before_norm = batch.return_hidden_states_before_norm
        batch.return_hidden_states_before_norm = False

        # capture_hidden_mode default: derive from SB.return_hidden_states /
        # spec_info.capture_hidden_mode when caller did not override.
        if capture_hidden_mode is None:
            if batch.return_hidden_states:
                capture_hidden_mode = CaptureHiddenMode.FULL
            elif batch.spec_info is not None:
                capture_hidden_mode = getattr(
                    batch.spec_info, "capture_hidden_mode", CaptureHiddenMode.NULL
                )
            else:
                capture_hidden_mode = CaptureHiddenMode.NULL

        # extend-mode-only fields are None on decode/idle
        if batch.forward_mode.is_decode_or_idle():
            extend_seq_lens = extend_prefix_lens = extend_logprob_start_lens = None
        else:
            extend_seq_lens = batch.extend_lens
            extend_prefix_lens = batch.prefix_lens
            extend_logprob_start_lens = batch.extend_logprob_start_lens

        # Mirror the grammars-population behavior previously done in
        # ScheduleBatch.get_model_worker_batch.
        if batch.sampling_info is not None:
            if batch.has_grammar:
                batch.sampling_info.grammars = [req.grammar for req in batch.reqs]
            else:
                batch.sampling_info.grammars = None

        # ScheduleBatch.sampling_info is already swapped to the forward-only
        # copy by Scheduler.run_batch under overlap mode (see save/restore
        # block there). Use it directly.
        if seq_lens_cpu_cache is not None:
            # Stale-cache guard: shape must match current GPU seq_lens. Mismatch
            # means caller forgot to refresh the override after batch size
            # changed (e.g. filter/merge_batch); using a stale cache would
            # propagate wrong CPU mirror to downstream DP / cudagraph logic.
            assert seq_lens_cpu_cache.shape == batch.seq_lens.shape, (
                f"seq_lens_cpu_cache shape {seq_lens_cpu_cache.shape} != "
                f"seq_lens {batch.seq_lens.shape}; stale override on batch?"
            )
            seq_lens_cpu = seq_lens_cpu_cache
        else:
            seq_lens_cpu = batch.seq_lens_cpu

```
**EN:** This callable implements `ForwardBatch.init_new`. It takes `cls`, `batch`, `model_runner` and mainly implements init new. This chunk is part 1 of 4 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; handles grammar or regular-expression constraints.
**CN:** 这一可调用对象实现了 `ForwardBatch.init_new`。它接收 `cls`, `batch`, `model_runner`，主要用于实现 init new 相关逻辑。 该片段是同一逻辑块的第 1/4 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；处理语法或正则约束。

### Lines 502-561: Method ForwardBatch.init_new (part 2/4)
```python
        ret = cls(
            forward_mode=batch.forward_mode,
            batch_size=len(batch.seq_lens),
            input_ids=batch.input_ids,
            req_pool_indices=batch.req_pool_indices,
            seq_lens=batch.seq_lens,
            out_cache_loc=batch.out_cache_loc,
            mamba_track_indices=batch.mamba_track_indices,
            mamba_track_mask=batch.mamba_track_mask,
            mamba_track_seqlens=batch.mamba_track_seqlens,
            mm_inputs=batch.multimodal_inputs,
            encoder_cached=batch.encoder_cached,
            encoder_lens=batch.encoder_lens,
            encoder_lens_cpu=batch.encoder_lens_cpu,
            encoder_out_cache_loc=batch.encoder_out_cache_loc,
            seq_lens_sum=batch.seq_lens_sum,
            seq_lens_cpu=seq_lens_cpu,
            orig_seq_lens=batch.orig_seq_lens,
            return_logprob=batch.return_logprob,
            top_logprobs_nums=batch.top_logprobs_nums,
            token_ids_logprobs=batch.token_ids_logprobs,
            is_extend_in_batch=batch.is_extend_in_batch,
            all_extend_in_batch=batch.all_extend_in_batch,
            can_run_dp_cuda_graph=batch.can_run_dp_cuda_graph,
            global_forward_mode=batch.global_forward_mode,
            is_prefill_only=batch.is_prefill_only,
            multi_item_delimiter_indices=batch.multi_item_delimiter_indices,
            lora_ids=[req.lora_id for req in batch.reqs],
            sampling_info=batch.sampling_info,
            req_to_token_pool=model_runner.req_to_token_pool,
            token_to_kv_pool=model_runner.token_to_kv_pool,
            attn_backend=model_runner.attn_backend,
            spec_algorithm=batch.spec_algorithm,
            spec_info=batch.spec_info,
            capture_hidden_mode=capture_hidden_mode,
            input_embeds=batch.input_embeds,
            replace_embeds=batch.replace_embeds,
            replace_positions=batch.replace_positions,
            token_type_ids=batch.token_type_ids,
            tbo_split_seq_index=batch.tbo_split_seq_index,
            dimensions=batch.dimensions,
            return_pooled_hidden_states=batch.return_pooled_hidden_states,
            return_hidden_states_before_norm=return_hidden_states_before_norm,
            rids=[req.rid for req in batch.reqs],
        )
        device = model_runner.device

        if batch.extend_input_logprob_token_ids is not None:
            ret.extend_input_logprob_token_ids_gpu = (
                batch.extend_input_logprob_token_ids.to(device, non_blocking=True)
            )

        num_tokens = len(batch.input_ids) if batch.input_ids is not None else 0
        if enable_num_token_non_padded():
            ret.num_token_non_padded = torch.tensor(num_tokens, dtype=torch.int32).to(
                device, non_blocking=True
            )
        ret.num_token_non_padded_cpu = num_tokens

        # For MLP sync
```
**EN:** This callable implements `ForwardBatch.init_new`. It takes `cls`, `batch`, `model_runner` and mainly implements init new. This chunk is part 2 of 4 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `ForwardBatch.init_new`。它接收 `cls`, `batch`, `model_runner`，主要用于实现 init new 相关逻辑。 该片段是同一逻辑块的第 2/4 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 562-621: Method ForwardBatch.init_new (part 3/4)
```python
        if batch.global_num_tokens is not None:
            assert batch.global_num_tokens_for_logprob is not None

            # process global_num_tokens and global_num_tokens_for_logprob
            if batch.spec_info is not None:
                spec_info: SpecInput = batch.spec_info
                global_num_tokens, global_num_tokens_for_logprob = (
                    spec_info.get_spec_adjusted_global_num_tokens(batch)
                )
            else:
                global_num_tokens = batch.global_num_tokens
                global_num_tokens_for_logprob = batch.global_num_tokens_for_logprob

            ret.original_global_num_tokens_cpu = batch.global_num_tokens
            ret.global_num_tokens_cpu = global_num_tokens
            ret.global_num_tokens_gpu = torch.tensor(
                global_num_tokens, dtype=torch.int64
            ).to(device, non_blocking=True)

            ret.global_num_tokens_for_logprob_cpu = global_num_tokens_for_logprob
            ret.global_num_tokens_for_logprob_gpu = torch.tensor(
                global_num_tokens_for_logprob, dtype=torch.int64
            ).to(device, non_blocking=True)

        if ret.forward_mode.is_idle():
            ret.positions = torch.empty((0,), dtype=torch.int64, device=device)
            return ret

        # Override the positions with diffusion LLM or spec_info
        if batch.dllm_config is not None:
            block_size = batch.dllm_config.block_size
            # Use int64 for AMD rotary embedding kernel compatibility
            positions_dtype = torch.int64 if is_hip() or _is_npu else torch.int32
            ret.positions = torch.tensor(
                [
                    i
                    for block_offset in (req.dllm_block_offset for req in batch.reqs)
                    for i in range(block_offset, block_offset + block_size)
                ],
                dtype=positions_dtype,
            ).to(device, non_blocking=True)
        elif (
            ret.spec_info is not None
            and getattr(ret.spec_info, "positions", None) is not None
        ):
            ret.positions = ret.spec_info.positions

        # Init position information
        if ret.forward_mode.is_decode() or ret.forward_mode.is_target_verify():
            if ret.positions is None:
                ret.positions = clamp_position(batch.seq_lens)
        else:
            assert isinstance(extend_seq_lens, list)
            assert isinstance(extend_prefix_lens, list)
            ret.extend_seq_lens = torch.tensor(extend_seq_lens, dtype=torch.int32).to(
                device, non_blocking=True
            )
            ret.extend_prefix_lens = torch.tensor(
                extend_prefix_lens, dtype=torch.int32
            ).to(device, non_blocking=True)
```
**EN:** This callable implements `ForwardBatch.init_new`. It takes `cls`, `batch`, `model_runner` and mainly implements init new. This chunk is part 3 of 4 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ForwardBatch.init_new`。它接收 `cls`, `batch`, `model_runner`，主要用于实现 init new 相关逻辑。 该片段是同一逻辑块的第 3/4 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 622-664: Method ForwardBatch.init_new (part 4/4)
```python
            ret.extend_num_tokens = batch.extend_num_tokens
            positions, ret.extend_start_loc = compute_position(
                model_runner.server_args.attention_backend,
                ret.extend_prefix_lens,
                ret.extend_seq_lens,
                ret.extend_num_tokens,
            )
            if ret.positions is None:
                ret.positions = positions
            ret.extend_prefix_lens_cpu = extend_prefix_lens
            ret.extend_seq_lens_cpu = extend_seq_lens
            ret.extend_logprob_start_lens_cpu = extend_logprob_start_lens

        if model_runner.use_ngram_embedding:
            ret._init_ngram_embedding_info(batch, device)

        if model_runner.model_is_mrope:
            if (
                ret.spec_info is not None
                and getattr(ret.spec_info, "positions", None) is not None
            ):
                ret.compute_spec_mrope_positions(model_runner, batch)
            else:
                ret._compute_mrope_positions(model_runner, batch)

        # Precompute SWA cache location once for all SWA layers
        if model_runner.is_hybrid_swa and ret.out_cache_loc is not None:
            ret.out_cache_loc_swa = (
                model_runner.token_to_kv_pool_allocator.translate_loc_from_full_to_swa(
                    ret.out_cache_loc
                )
            )

        # Init lora information
        if model_runner.server_args.enable_lora:
            # In the non-LoRA overlap loading case, we fetch LoRA adapters into the memory pool
            # as a batch, right before running the batch
            if not model_runner.server_args.enable_lora_overlap_loading:
                model_runner.lora_manager.fetch_new_loras(set(ret.lora_ids))

            model_runner.lora_manager.prepare_lora_batch(ret)

        return ret
```
**EN:** This callable implements `ForwardBatch.init_new`. It takes `cls`, `batch`, `model_runner` and mainly implements init new. This chunk is part 4 of 4 for the same logical block.
**CN:** 这一可调用对象实现了 `ForwardBatch.init_new`。它接收 `cls`, `batch`, `model_runner`，主要用于实现 init new 相关逻辑。 该片段是同一逻辑块的第 4/4 部分。

### Lines 666-681: Method ForwardBatch.adjust_num_token_non_padded_for_attn_tp
```python
    def adjust_num_token_non_padded_for_attn_tp(self, server_args) -> None:
        """Make num_token_non_padded local to this attention-TP rank."""
        from sglang.srt.utils.common import require_mlp_tp_gather

        dp_rank = get_attention_dp_rank()
        assert self.global_num_tokens_cpu is not None

        if require_mlp_tp_gather(server_args):
            num_tokens_per_dp = self.global_num_tokens_cpu[dp_rank]
        else:
            num_tokens_per_dp = self.global_num_tokens_cpu[0]

        self.num_token_non_padded = compute_local_num_token_non_padded(
            global_num_token_non_padded=self.num_token_non_padded,
            num_tokens_per_dp=num_tokens_per_dp,
        )
```
**EN:** This callable implements `ForwardBatch.adjust_num_token_non_padded_for_attn_tp`. It takes `server_args` and mainly adds configuration entries or arguments. The docstring states: "Make num_token_non_padded local to this attention-TP rank." In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ForwardBatch.adjust_num_token_non_padded_for_attn_tp`。它接收 `server_args`，主要用于添加配置项或参数。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 683-704: Method ForwardBatch.merge_mm_inputs
```python
    def merge_mm_inputs(self) -> Optional[MultimodalInputs]:
        """
        Merge all multimodal inputs in the batch into a single MultiModalInputs object.

        Returns:
            if none, current batch contains no multimodal input

        """
        if not self.mm_inputs or all(x is None for x in self.mm_inputs):
            return None
        # Filter out None values
        valid_inputs = [x for x in self.mm_inputs if x is not None]

        # TODO: is it expensive?
        # a workaround to avoid importing `MultimodalInputs`
        merged = valid_inputs[0].__class__(mm_items=[])

        # Merge remaining inputs
        for mm_input in valid_inputs:
            merged.merge(mm_input)

        return merged
```
**EN:** This callable implements `ForwardBatch.merge_mm_inputs` and mainly merges related state. The docstring states: "Merge all multimodal inputs in the batch into a single MultiModalInputs object."
**CN:** 这一可调用对象实现了 `ForwardBatch.merge_mm_inputs`，主要用于合并相关状态。

### Lines 706-712: Method ForwardBatch.contains_image_inputs
```python
    def contains_image_inputs(self) -> bool:
        if self.mm_inputs is None:
            return False
        return any(
            mm_input is not None and mm_input.contains_image_inputs()
            for mm_input in self.mm_inputs
        )
```
**EN:** This callable implements `ForwardBatch.contains_image_inputs` and mainly implements contains image inputs.
**CN:** 这一可调用对象实现了 `ForwardBatch.contains_image_inputs`，主要用于实现 contains image inputs 相关逻辑。

### Lines 714-720: Method ForwardBatch.contains_audio_inputs
```python
    def contains_audio_inputs(self) -> bool:
        if self.mm_inputs is None:
            return False
        return any(
            mm_input is not None and mm_input.contains_audio_inputs()
            for mm_input in self.mm_inputs
        )
```
**EN:** This callable implements `ForwardBatch.contains_audio_inputs` and mainly implements contains audio inputs.
**CN:** 这一可调用对象实现了 `ForwardBatch.contains_audio_inputs`，主要用于实现 contains audio inputs 相关逻辑。

### Lines 722-728: Method ForwardBatch.contains_video_inputs
```python
    def contains_video_inputs(self) -> bool:
        if self.mm_inputs is None:
            return False
        return any(
            mm_input is not None and mm_input.contains_video_inputs()
            for mm_input in self.mm_inputs
        )
```
**EN:** This callable implements `ForwardBatch.contains_video_inputs` and mainly implements contains video inputs.
**CN:** 这一可调用对象实现了 `ForwardBatch.contains_video_inputs`，主要用于实现 contains video inputs 相关逻辑。

### Lines 730-735: Method ForwardBatch.contains_mm_inputs
```python
    def contains_mm_inputs(self) -> bool:
        return (
            self.contains_audio_inputs()
            or self.contains_video_inputs()
            or self.contains_image_inputs()
        )
```
**EN:** This callable implements `ForwardBatch.contains_mm_inputs` and mainly implements contains mm inputs.
**CN:** 这一可调用对象实现了 `ForwardBatch.contains_mm_inputs`，主要用于实现 contains mm inputs 相关逻辑。

### Lines 737-748: Method ForwardBatch._init_ngram_embedding_info
```python
    def _init_ngram_embedding_info(self, batch: ScheduleBatch, device: torch.device):
        if self.forward_mode.is_decode():
            column_starts, req_lens = self.seq_lens - 1, 1
        else:
            column_starts, req_lens = self.extend_prefix_lens, self.extend_seq_lens
        self.ngram_embedding_info = NgramEmbeddingInfo.create(
            batch.ne_token_table,
            self.batch_size,
            device,
            column_starts=column_starts,
            req_lens=req_lens,
        )
```
**EN:** This callable implements `ForwardBatch._init_ngram_embedding_info`. It takes `batch`, `device` and mainly implements init ngram embedding info.
**CN:** 这一可调用对象实现了 `ForwardBatch._init_ngram_embedding_info`。它接收 `batch`, `device`，主要用于实现 init ngram embedding info 相关逻辑。

### Lines 750-800: Method ForwardBatch.compute_spec_mrope_positions
```python
    def compute_spec_mrope_positions(
        self, model_runner: ModelRunner, batch: ScheduleBatch
    ):
        # TODO support batched deltas
        batch_size = self.seq_lens.shape[0]
        device = model_runner.device
        mm_inputs = batch.multimodal_inputs

        if batch.forward_mode.is_draft_extend():  # draft_extend_after_decode
            mrope_deltas = []
            extend_lens = []
            for batch_idx in range(batch_size):
                extend_seq_len = batch.extend_lens[batch_idx]
                extend_lens.append(extend_seq_len)
                mrope_delta = (
                    torch.zeros(1, dtype=torch.int64)
                    if mm_inputs[batch_idx] is None
                    else mm_inputs[batch_idx].mrope_position_delta.squeeze(0)
                )
                mrope_deltas.append(mrope_delta.to(device=device))
            position_chunks = torch.split(batch.spec_info.positions, extend_lens)
            mrope_positions_list = [
                pos_chunk + delta
                for pos_chunk, delta in zip(position_chunks, mrope_deltas)
            ]
            next_input_positions = (
                torch.cat(mrope_positions_list, dim=0).unsqueeze(0).repeat(3, 1)
            )

        else:  # target_verify or draft_decode
            seq_positions = batch.spec_info.positions.view(batch_size, -1)
            # Split text-only and mixed batches here because SpecV2 text-only batches can avoid an extra D2H.
            if all(mm_input is None for mm_input in mm_inputs):
                mrope_delta_tensor = torch.zeros(
                    (batch_size, 1), dtype=torch.int64, device=device
                )
            else:
                mrope_deltas = [
                    (
                        torch.zeros(1, dtype=torch.int64)
                        if mm_inputs[i] is None
                        else mm_inputs[i].mrope_position_delta.squeeze(0)
                    )
                    for i in range(batch_size)
                ]
                mrope_delta_tensor = torch.stack(mrope_deltas, dim=0).to(device=device)
            next_input_positions = (
                (seq_positions + mrope_delta_tensor).flatten().unsqueeze(0).repeat(3, 1)
            )

        self.mrope_positions = next_input_positions
```
**EN:** This callable implements `ForwardBatch.compute_spec_mrope_positions`. It takes `model_runner`, `batch` and mainly implements compute spec mrope positions.
**CN:** 这一可调用对象实现了 `ForwardBatch.compute_spec_mrope_positions`。它接收 `model_runner`, `batch`，主要用于实现 compute spec mrope positions 相关逻辑。

### Lines 802-813: Method ForwardBatch._expand_mrope_from_input
```python
    def _expand_mrope_from_input(
        self,
        mm_input: MultimodalInputs,
        seq_len: int,
    ) -> torch.Tensor:
        # doing below compute on cpu to avoid frequent small kernels
        if mm_input.mrope_position_delta_repeated_cache is None:
            mm_input.mrope_position_delta_repeated_cache = (
                (mm_input.mrope_position_delta - 1).flatten().unsqueeze(0).repeat(3, 1)
            )
        mrope_positions = mm_input.mrope_position_delta_repeated_cache + seq_len
        return mrope_positions
```
**EN:** This callable implements `ForwardBatch._expand_mrope_from_input`. It takes `mm_input`, `seq_len` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `ForwardBatch._expand_mrope_from_input`。它接收 `mm_input`, `seq_len`，主要用于从外部表示构造数据。

### Lines 815-873: Method ForwardBatch._compute_mrope_positions
```python
    def _compute_mrope_positions(self, model_runner: ModelRunner, batch: ScheduleBatch):
        # batch_size * [3 * seq_len]
        batch_size = self.seq_lens_cpu.shape[0]
        mrope_positions_list = [[]] * batch_size
        for batch_idx in range(batch_size):
            mm_input = batch.multimodal_inputs[batch_idx]
            if self.forward_mode.is_decode():
                # 3 * N
                if (
                    mm_input is None
                    or get_global_server_args().rl_on_policy_target is not None
                ):
                    mrope_positions_list[batch_idx] = torch.full(
                        (3, 1),
                        self.seq_lens_cpu[batch_idx] - 1,
                        dtype=torch.int64,
                    )
                else:
                    mrope_positions = self._expand_mrope_from_input(
                        mm_input, self.seq_lens_cpu[batch_idx]
                    )
                    mrope_positions_list[batch_idx] = mrope_positions
            elif self.forward_mode.is_extend(include_draft_extend_v2=True):
                extend_seq_len, extend_prefix_len = (
                    batch.extend_lens[batch_idx],
                    batch.prefix_lens[batch_idx],
                )
                if (
                    mm_input is None
                    or get_global_server_args().rl_on_policy_target is not None
                ):
                    # text only
                    mrope_positions = torch.tensor(
                        [
                            [
                                pos
                                for pos in range(
                                    extend_prefix_len,
                                    extend_prefix_len + extend_seq_len,
                                )
                            ]
                        ]
                        * 3
                    )
                else:
                    mrope_positions = mm_input.mrope_positions[
                        :,
                        extend_prefix_len : extend_prefix_len + extend_seq_len,
                    ]
                    if mrope_positions.numel() == 0:
                        mrope_positions = self._expand_mrope_from_input(
                            mm_input, self.seq_lens_cpu[batch_idx]
                        )
                mrope_positions_list[batch_idx] = mrope_positions

        self.mrope_positions = torch.cat(
            [pos for pos in mrope_positions_list],
            dim=1,
        ).to(dtype=torch.int64, device=model_runner.device, non_blocking=True)
```
**EN:** This callable implements `ForwardBatch._compute_mrope_positions`. It takes `model_runner`, `batch` and mainly implements compute mrope positions.
**CN:** 这一可调用对象实现了 `ForwardBatch._compute_mrope_positions`。它接收 `model_runner`, `batch`，主要用于实现 compute mrope positions 相关逻辑。

### Lines 875-888: Method ForwardBatch._pad_tensor_to_size
```python
    def _pad_tensor_to_size(self, tensor: torch.Tensor, size: int, *, value: int = 0):
        if value == 0:
            return torch.cat(
                [tensor, tensor.new_zeros(size - tensor.shape[0], *tensor.shape[1:])],
                dim=0,
            )
        else:
            return torch.cat(
                [
                    tensor,
                    tensor.new_full((size - tensor.shape[0], *tensor.shape[1:]), value),
                ],
                dim=0,
            )
```
**EN:** This callable implements `ForwardBatch._pad_tensor_to_size`. It takes `tensor`, `size` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `ForwardBatch._pad_tensor_to_size`。它接收 `tensor`, `size`，主要用于将数据转换为另一种表示。

### Lines 890-949: Method ForwardBatch.prepare_mlp_sync_batch (part 1/2)
```python
    def prepare_mlp_sync_batch(self, model_runner: ModelRunner):
        from sglang.srt.batch_overlap.two_batch_overlap import TboForwardBatchPreparer

        assert self.global_num_tokens_cpu is not None
        assert self.global_num_tokens_for_logprob_cpu is not None

        global_num_tokens = self.global_num_tokens_cpu
        sync_group_size = len(global_num_tokens)
        attn_tp_size = get_attention_tp_size()

        for i in range(sync_group_size):
            # make sure that the padded length is divisible by attn_tp_size because we may need reduce-scatter across attn_tp dim.
            # there is no reduce-scatter in LM logprob, so we do not need to adjust the padded length for logprob
            global_num_tokens[i] = ceil_align(global_num_tokens[i], attn_tp_size)

        # make sure that each rank has the same number of tokens to do collective communication.
        attn_cp_size = get_attention_cp_size()
        for i in range(sync_group_size):
            global_num_tokens[i] = ceil_align(global_num_tokens[i], attn_cp_size)

        dp_padding_mode = DpPaddingMode.get_dp_padding_mode(
            self.is_extend_in_batch, global_num_tokens
        )
        self.dp_padding_mode = dp_padding_mode

        if dp_padding_mode.is_max_len():
            # when DP gather mode is all gather, we will use
            # all_gather_into_tensor to gather hidden states, where transferred
            # tokens should be padded to the same length. We will also use
            # reduce-scatter instead of all-reduce after MLP.
            max_num_tokens = max(global_num_tokens)
            global_num_tokens = [max_num_tokens] * sync_group_size
            buffer_len = max_num_tokens * sync_group_size
        else:
            buffer_len = sum(global_num_tokens)

        if len(global_num_tokens) > 1:
            num_tokens = global_num_tokens[get_attention_dp_rank()]
        else:
            num_tokens = global_num_tokens[0]

        self.global_dp_buffer_len = buffer_len
        set_dp_buffer_len(
            buffer_len, num_tokens, dp_padding_mode.is_max_len(), global_num_tokens
        )
        set_is_extend_in_batch(self.is_extend_in_batch)

        bs = self.batch_size

        if (
            self.forward_mode.is_decode()
            or self.forward_mode.is_target_verify()
            or self.forward_mode.is_draft_extend(include_v2=True)
            or self.forward_mode.is_idle()
        ):
            if self.is_extend_in_batch and dp_padding_mode.is_max_len():
                setattr(self, "_original_forward_mode", self.forward_mode)
                self.forward_mode = ForwardMode.EXTEND
                self.extend_num_tokens = bs
                self.extend_seq_lens = torch.full_like(self.seq_lens, 1)
```
**EN:** This callable implements `ForwardBatch.prepare_mlp_sync_batch`. It takes `model_runner` and mainly prepares runtime inputs. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ForwardBatch.prepare_mlp_sync_batch`。它接收 `model_runner`，主要用于准备运行时输入。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 950-984: Method ForwardBatch.prepare_mlp_sync_batch (part 2/2)
```python
                self.extend_prefix_lens = self.seq_lens - 1
                self.extend_start_loc = torch.arange(
                    bs, dtype=torch.int32, device=self.seq_lens.device
                )
                self.extend_prefix_lens_cpu = self.extend_prefix_lens.cpu()
                self.extend_seq_lens_cpu = self.extend_seq_lens.cpu()
                self.extend_logprob_start_lens_cpu = self.extend_prefix_lens_cpu
            else:
                setattr(self, "_original_batch_size", self.batch_size)
                if self.spec_info is not None:
                    bs = self.batch_size = (
                        num_tokens // self.spec_info.num_tokens_per_req
                    )
                else:
                    bs = self.batch_size = num_tokens
        elif self.forward_mode.is_extend():
            self.extend_num_tokens = num_tokens

        # padding
        self._pad_inputs_to_size(model_runner, num_tokens, bs)
        self.global_num_tokens_cpu = global_num_tokens
        global_num_tokens_pinned = torch.tensor(global_num_tokens, pin_memory=True)
        self.global_num_tokens_gpu.copy_(global_num_tokens_pinned, non_blocking=True)

        TboForwardBatchPreparer.prepare(
            batch=self, is_draft_worker=model_runner.is_draft_worker
        )
        # TODO: The following is added to make sure sub-batch input_ids are padded
        # to the multiple of attn_tp_size. It can likely be removed after this
        # function is refactored and merged into the Scheduler.
        if self.tbo_children:
            for child in self.tbo_children:
                child._pad_inputs_to_size(
                    model_runner, child.tbo_padded_len, child.batch_size
                )
```
**EN:** This callable implements `ForwardBatch.prepare_mlp_sync_batch`. It takes `model_runner` and mainly prepares runtime inputs. This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `ForwardBatch.prepare_mlp_sync_batch`。它接收 `model_runner`，主要用于准备运行时输入。 该片段是同一逻辑块的第 2/2 部分。

### Lines 986-1045: Method ForwardBatch._pad_inputs_to_size (part 1/2)
```python
    def _pad_inputs_to_size(self, model_runner: ModelRunner, num_tokens, bs):
        # padding
        self.input_ids = self._pad_tensor_to_size(self.input_ids, num_tokens)
        self.req_pool_indices = self._pad_tensor_to_size(self.req_pool_indices, bs)
        self.lora_ids.extend((bs - len(self.lora_ids)) * [None])

        seq_len_fill_value = (
            model_runner.attn_backend.get_cuda_graph_seq_len_fill_value()
        )
        self.seq_lens_sum = self.seq_lens_sum + seq_len_fill_value * (
            bs - self.seq_lens.shape[0]
        )
        self.seq_lens = self._pad_tensor_to_size(
            self.seq_lens, bs, value=seq_len_fill_value
        )
        if self.seq_lens_cpu is not None:
            self.seq_lens_cpu = self._pad_tensor_to_size(
                self.seq_lens_cpu, bs, value=seq_len_fill_value
            )

        self.out_cache_loc = self._pad_tensor_to_size(self.out_cache_loc, num_tokens)
        if self.out_cache_loc_swa is not None:
            self.out_cache_loc_swa = self._pad_tensor_to_size(
                self.out_cache_loc_swa, num_tokens
            )
        if self.encoder_lens is not None:
            self.encoder_lens = self._pad_tensor_to_size(self.encoder_lens, bs)
        self.positions = self._pad_tensor_to_size(self.positions, num_tokens)
        if self.mamba_track_indices is not None:
            self.mamba_track_indices = self._pad_tensor_to_size(
                self.mamba_track_indices, bs
            )
        if self.mamba_track_mask is not None:
            self.mamba_track_mask = self._pad_tensor_to_size(self.mamba_track_mask, bs)
        if self.mamba_track_seqlens is not None:
            self.mamba_track_seqlens = self._pad_tensor_to_size(
                self.mamba_track_seqlens, bs
            )

        if self.mrope_positions is not None:
            self.mrope_positions = torch.cat(
                [
                    self.mrope_positions,
                    self.mrope_positions.new_zeros(
                        3, num_tokens - self.mrope_positions.shape[1]
                    ),
                ],
                dim=1,
            )

        # TODO: check if we need to pad other tensors
        if self.extend_seq_lens is not None:
            self.extend_seq_lens = self._pad_tensor_to_size(self.extend_seq_lens, bs)

        if self.spec_info is not None and self.spec_info.is_draft_input():
            spec_info = self.spec_info
            self.output_cache_loc_backup = self.out_cache_loc
            self.hidden_states_backup = spec_info.hidden_states
            # spec_info is EagleDraftInput | EagleDraftExtendInput; each carries
            # a disjoint subset of the fields below, so getattr-guard each one.
```
**EN:** This callable implements `ForwardBatch._pad_inputs_to_size`. It takes `model_runner`, `num_tokens`, `bs` and mainly converts data into another representation. This chunk is part 1 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `ForwardBatch._pad_inputs_to_size`。它接收 `model_runner`, `num_tokens`, `bs`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1046-1061: Method ForwardBatch._pad_inputs_to_size (part 2/2)
```python
            if getattr(spec_info, "topk_p", None) is not None:
                spec_info.topk_p = self._pad_tensor_to_size(spec_info.topk_p, bs)
            if getattr(spec_info, "topk_index", None) is not None:
                spec_info.topk_index = self._pad_tensor_to_size(
                    spec_info.topk_index, bs
                )
            if getattr(spec_info, "num_correct_drafts", None) is not None:
                spec_info.num_correct_drafts = self._pad_tensor_to_size(
                    spec_info.num_correct_drafts, bs
                )
                spec_info.num_accept_tokens = self._pad_tensor_to_size(
                    spec_info.num_accept_tokens, bs
                )
            spec_info.hidden_states = self._pad_tensor_to_size(
                spec_info.hidden_states, num_tokens
            )
```
**EN:** This callable implements `ForwardBatch._pad_inputs_to_size`. It takes `model_runner`, `num_tokens`, `bs` and mainly converts data into another representation. This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `ForwardBatch._pad_inputs_to_size`。它接收 `model_runner`, `num_tokens`, `bs`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。

### Lines 1063-1074: Method ForwardBatch.prepare_attn_tp_scatter_input
```python
    def prepare_attn_tp_scatter_input(self, model_runner: ModelRunner):
        from sglang.srt.layers.communicator import get_attn_tp_context

        attn_tp_context = get_attn_tp_context()
        input_scattered = attn_tp_context.use_input_scattered(self)
        if not input_scattered:
            return
        assert self.forward_mode.is_extend()
        tokens = self.input_ids.shape[0]
        rank_size = get_tensor_model_parallel_world_size()
        tokens_padded = (tokens + rank_size - 1) // rank_size * rank_size
        self._pad_inputs_to_size(model_runner, tokens_padded, self.batch_size)
```
**EN:** This callable implements `ForwardBatch.prepare_attn_tp_scatter_input`. It takes `model_runner` and mainly prepares runtime inputs. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ForwardBatch.prepare_attn_tp_scatter_input`。它接收 `model_runner`，主要用于准备运行时输入。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 1076-1130: Method ForwardBatch.post_forward_mlp_sync_batch
```python
    def post_forward_mlp_sync_batch(self, logits_output: LogitsProcessorOutput):

        self.forward_mode = getattr(self, "_original_forward_mode", self.forward_mode)
        self.batch_size = getattr(self, "_original_batch_size", self.batch_size)
        bs = self.batch_size

        if self.spec_info is not None:
            if self.forward_mode.is_decode():  # draft
                num_tokens = self.hidden_states_backup.shape[0]
                self.positions = self.positions[:num_tokens]
                self.seq_lens = self.seq_lens[:bs]
                self.req_pool_indices = self.req_pool_indices[:bs]
                if self.seq_lens_cpu is not None:
                    self.seq_lens_cpu = self.seq_lens_cpu[:bs]
                logits_output.next_token_logits = logits_output.next_token_logits[
                    :num_tokens
                ]
                logits_output.hidden_states = logits_output.hidden_states[:num_tokens]
            elif self.forward_mode.is_target_verify():  # verify
                num_tokens = bs * self.spec_info.draft_token_num
                logits_output.next_token_logits = logits_output.next_token_logits[
                    :num_tokens
                ]
                logits_output.hidden_states = logits_output.hidden_states[:num_tokens]
            elif self.forward_mode.is_draft_extend():  # draft extend
                self.spec_info.num_correct_drafts = self.spec_info.num_correct_drafts[
                    :bs
                ]
                self.spec_info.num_accept_tokens = self.spec_info.num_accept_tokens[:bs]
                logits_output.next_token_logits = logits_output.next_token_logits[:bs]
                logits_output.hidden_states = logits_output.hidden_states[:bs]
            elif self.forward_mode.is_draft_extend_v2():  # draft extend_v2
                bs = bs * self.spec_info.num_tokens_per_req
                logits_output.next_token_logits = logits_output.next_token_logits[:bs]
                logits_output.hidden_states = logits_output.hidden_states[:bs]
            elif self.forward_mode.is_extend() or self.forward_mode.is_idle():
                logits_output.next_token_logits = logits_output.next_token_logits[:bs]
                logits_output.hidden_states = logits_output.hidden_states[:bs]

            if hasattr(self, "hidden_states_backup"):
                self.spec_info.hidden_states = self.hidden_states_backup
            if hasattr(self, "output_cache_loc_backup"):
                self.out_cache_loc = self.output_cache_loc_backup

        elif self.forward_mode.is_decode() or self.forward_mode.is_idle():
            logits_output.next_token_logits = logits_output.next_token_logits[:bs]
            if logits_output.hidden_states is not None:
                logits_output.hidden_states = logits_output.hidden_states[:bs]
        elif self.forward_mode.is_extend():
            num_tokens = self.seq_lens_sum
            logits_output.next_token_logits = logits_output.next_token_logits[
                :num_tokens
            ]
            if logits_output.hidden_states is not None:
                logits_output.hidden_states = logits_output.hidden_states[:num_tokens]
```
**EN:** This callable implements `ForwardBatch.post_forward_mlp_sync_batch`. It takes `logits_output` and mainly implements post forward mlp sync batch.
**CN:** 这一可调用对象实现了 `ForwardBatch.post_forward_mlp_sync_batch`。它接收 `logits_output`，主要用于实现 post forward mlp sync batch 相关逻辑。

### Lines 1132-1134: Method ForwardBatch.can_run_tbo
```python
    @property
    def can_run_tbo(self):
        return self.tbo_split_seq_index is not None
```
**EN:** This callable implements `ForwardBatch.can_run_tbo` and mainly executes the main workflow.
**CN:** 这一可调用对象实现了 `ForwardBatch.can_run_tbo`，主要用于执行主要流程。

### Lines 1137-1138: Function enable_num_token_non_padded
```python
def enable_num_token_non_padded():
    return get_moe_expert_parallel_world_size() > 1
```
**EN:** This callable implements `enable_num_token_non_padded` and mainly adds configuration entries or arguments.
**CN:** 这一可调用对象实现了 `enable_num_token_non_padded`，主要用于添加配置项或参数。

### Lines 1141-1144: Class PPProxyTensors
```python
class PPProxyTensors:
    # adapted from https://github.com/vllm-project/vllm/blob/d14e98d924724b284dc5eaf8070d935e214e50c0/vllm/sequence.py#L1103
    tensors: Dict[str, torch.Tensor]

```
**EN:** This range introduces `PPProxyTensors` and defines the structure or metadata that its methods rely on. In this range it sets up imports and shared symbols.
**CN:** 这一段引入 `PPProxyTensors`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1145-1150: Method PPProxyTensors.__init__
```python
    def __init__(self, tensors):
        # manually define this function, so that
        # Dynamo knows `IntermediateTensors()` comes from this file.
        # Otherwise, dataclass will generate this function by evaluating
        # a string, and we will lose the information about the source file.
        self.tensors = tensors
```
**EN:** This callable implements `PPProxyTensors.__init__`. It takes `tensors` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `PPProxyTensors.__init__`。它接收 `tensors`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1152-1156: Method PPProxyTensors.__getitem__
```python
    def __getitem__(self, key: Union[str, slice]):
        if isinstance(key, str):
            return self.tensors[key]
        elif isinstance(key, slice):
            return self.__class__({k: v[key] for k, v in self.tensors.items()})
```
**EN:** This callable implements `PPProxyTensors.__getitem__`. It takes `key` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `PPProxyTensors.__getitem__`。它接收 `key`，主要用于获取某个值或派生视图。

### Lines 1158-1159: Method PPProxyTensors.__setitem__
```python
    def __setitem__(self, key: str, value: torch.Tensor):
        self.tensors[key] = value
```
**EN:** This callable implements `PPProxyTensors.__setitem__`. It takes `key`, `value` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `PPProxyTensors.__setitem__`。它接收 `key`, `value`，主要用于将配置写入可变状态。

### Lines 1161-1162: Method PPProxyTensors.__len__
```python
    def __len__(self):
        return len(self.tensors)
```
**EN:** This callable implements `PPProxyTensors.__len__` and mainly implements len.
**CN:** 这一可调用对象实现了 `PPProxyTensors.__len__`，主要用于实现 len 相关逻辑。

### Lines 1164-1165: Method PPProxyTensors.__eq__
```python
    def __eq__(self, other: object):
        return isinstance(other, self.__class__) and self
```
**EN:** This callable implements `PPProxyTensors.__eq__`. It takes `other` and mainly implements eq.
**CN:** 这一可调用对象实现了 `PPProxyTensors.__eq__`。它接收 `other`，主要用于实现 eq 相关逻辑。

### Lines 1167-1168: Method PPProxyTensors.__repr__
```python
    def __repr__(self) -> str:
        return f"PPProxyTensors(tensors={self.tensors})"
```
**EN:** This callable implements `PPProxyTensors.__repr__` and mainly implements repr.
**CN:** 这一可调用对象实现了 `PPProxyTensors.__repr__`，主要用于实现 repr 相关逻辑。

### Lines 1171-1187: Function compute_position
```python
def compute_position(
    attn_backend: str,
    extend_prefix_lens: torch.Tensor,
    extend_seq_lens: torch.Tensor,
    extend_seq_lens_sum: int,
):
    if support_triton(attn_backend):
        positions, extend_start_loc = compute_position_triton(
            extend_prefix_lens,
            extend_seq_lens,
            extend_seq_lens_sum,
        )
    else:
        positions, extend_start_loc = compute_position_torch(
            extend_prefix_lens, extend_seq_lens
        )
    return positions, extend_start_loc
```
**EN:** This callable implements `compute_position`. It takes `attn_backend`, `extend_prefix_lens`, `extend_seq_lens`, `extend_seq_lens_sum` and mainly implements compute position.
**CN:** 这一可调用对象实现了 `compute_position`。它接收 `attn_backend`, `extend_prefix_lens`, `extend_seq_lens`, `extend_seq_lens_sum`，主要用于实现 compute position 相关逻辑。

### Lines 1190-1213: Function compute_position_triton
```python
def compute_position_triton(
    extend_prefix_lens: torch.Tensor, extend_seq_lens: torch.Tensor, extend_seq_lens_sum
):
    """Compute positions. It is a fused version of `compute_position_torch`."""
    batch_size = extend_seq_lens.shape[0]
    has_prefix = extend_prefix_lens.shape[0] == batch_size

    positions = torch.empty(
        extend_seq_lens_sum, dtype=torch.int64, device=extend_seq_lens.device
    )
    extend_start_loc = torch.empty(
        batch_size, dtype=torch.int32, device=extend_seq_lens.device
    )

    # Launch kernel
    compute_position_kernel[(batch_size,)](
        positions,
        extend_start_loc,
        extend_prefix_lens,
        extend_seq_lens,
        has_prefix,
    )

    return positions, extend_start_loc
```
**EN:** This callable implements `compute_position_triton`. It takes `extend_prefix_lens`, `extend_seq_lens`, `extend_seq_lens_sum` and mainly converts data into another representation. The docstring states: "Compute positions."
**CN:** 这一可调用对象实现了 `compute_position_triton`。它接收 `extend_prefix_lens`, `extend_seq_lens`, `extend_seq_lens_sum`，主要用于将数据转换为另一种表示。

### Lines 1216-1243: Function compute_position_kernel
```python
@triton.jit
def compute_position_kernel(
    positions,
    extend_start_loc,
    extend_prefix_lens,
    extend_seq_lens,
    has_prefix: tl.constexpr,
):
    BLOCK_SIZE: tl.constexpr = 512
    pid = tl.program_id(0).to(tl.int64)

    prefix_len = tl.load(extend_prefix_lens + pid) if has_prefix else 0
    seq_len = tl.load(extend_seq_lens + pid)

    # NOTE: This can be slow for large bs
    cumsum_start = tl.cast(0, tl.int64)
    for i in range(pid):
        cumsum_start += tl.load(extend_seq_lens + i)

    num_loop = tl.cdiv(seq_len, BLOCK_SIZE)
    for i in range(num_loop):
        offset = tl.arange(0, BLOCK_SIZE) + i * BLOCK_SIZE
        tl.store(
            positions + cumsum_start + offset,
            prefix_len + offset,
            mask=offset < seq_len,
        )
    tl.store(extend_start_loc + pid, cumsum_start)
```
**EN:** This callable implements `compute_position_kernel`. It takes `positions`, `extend_start_loc`, `extend_prefix_lens`, `extend_seq_lens` and mainly implements compute position kernel.
**CN:** 这一可调用对象实现了 `compute_position_kernel`。它接收 `positions`, `extend_start_loc`, `extend_prefix_lens`, `extend_seq_lens`，主要用于实现 compute position kernel 相关逻辑。

### Lines 1246-1260: Function compute_position_torch
```python
def compute_position_torch(
    extend_prefix_lens: torch.Tensor, extend_seq_lens: torch.Tensor
):
    positions = torch.cat(
        [
            torch.arange(
                prefix_len, prefix_len + extend_len, device=extend_prefix_lens.device
            )
            for prefix_len, extend_len in zip(extend_prefix_lens, extend_seq_lens)
        ],
        axis=0,
    )
    extend_start_loc = torch.zeros_like(extend_seq_lens)
    extend_start_loc[1:] = torch.cumsum(extend_seq_lens[:-1], dim=0)
    return positions.to(torch.int64), extend_start_loc
```
**EN:** This callable implements `compute_position_torch`. It takes `extend_prefix_lens`, `extend_seq_lens` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `compute_position_torch`。它接收 `extend_prefix_lens`, `extend_seq_lens`，主要用于将数据转换为另一种表示。

### Lines 1263-1264: Function _clamp_position_native
```python
def _clamp_position_native(seq_lens):
    return torch.clamp((seq_lens - 1), min=0).to(torch.int64)
```
**EN:** This callable implements `_clamp_position_native`. It takes `seq_lens` and mainly implements clamp position native.
**CN:** 这一可调用对象实现了 `_clamp_position_native`。它接收 `seq_lens`，主要用于实现 clamp position native 相关逻辑。

### Lines 1265-1272: Module imports, constants, and setup
```python


if is_cuda() or is_hip():
    from sglang.jit_kernel.clamp_position import clamp_position_cuda

    clamp_position = clamp_position_cuda
else:
    clamp_position = _clamp_position_native
```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

## Key Concepts / 关键概念
- `ForwardMode`: core class or state container / 核心类或状态容器
- `CaptureHiddenMode`: core class or state container / 核心类或状态容器
- `compute_local_num_token_non_padded`: adds configuration entries or arguments / 添加配置项或参数
- `NgramEmbeddingInfo`: core class or state container / 核心类或状态容器
- `ForwardBatch`: core class or state container / 核心类或状态容器
- `enable_num_token_non_padded`: adds configuration entries or arguments / 添加配置项或参数
- `PPProxyTensors`: core class or state container / 核心类或状态容器
- `compute_position`: implements compute position / 实现 compute position 相关逻辑
- `compute_position_triton`: converts data into another representation / 将数据转换为另一种表示
- `compute_position_kernel`: implements compute position kernel / 实现 compute position kernel 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `enum`, `functools`, `typing`
- **Third-party / 第三方**: `torch`, `triton`, `triton.language`
- **Internal modules / 内部模块**: `sglang.srt.distributed.parallel_state`, `sglang.srt.layers.dp_attention`, `sglang.srt.layers.utils.cp_utils`, `sglang.srt.model_executor.forward_batch_deepseek_mha_mixin`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.common`, `sglang.srt.layers.attention.base_attn_backend`, `sglang.srt.layers.logits_processor`, `sglang.srt.managers.hisparse_coordinator`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.memory_pool` + 6 more
