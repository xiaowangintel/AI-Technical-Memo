# logits_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/logits_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements sampling and logits post-processing for the SGLang SRT runtime. It exposes symbols such as `LogitsProcessorOutput`, `LogitsMetadata`, `LogitsProcessor`, and `fused_softcap_kernel` and connects them to backend-specific paths such as `CUDA`, `Triton`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了采样与 logits 后处理。它提供了 `LogitsProcessorOutput`、`LogitsMetadata`、`LogitsProcessor` 以及 `fused_softcap_kernel` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
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
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 14-65: Imports, constants, and runtime setup
```python
"""Logits processing."""

import dataclasses
import logging
from typing import Any, Dict, List, Optional, Tuple, Union

import torch
import triton
import triton.language as tl
from torch import nn
from triton.language.extra import libdevice

from sglang.srt.distributed import (
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_gather,
)
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import (
    DpPaddingMode,
    attn_tp_all_gather,
    attn_tp_all_gather_into_tensor,
    dp_gather_replicate,
    dp_scatter,
    get_attention_dp_rank,
    get_attention_dp_size,
    get_attention_tp_size,
    get_dp_device,
    get_dp_dtype,
    get_dp_hidden_size,
)
from sglang.srt.layers.utils.logprob import (
    InputLogprobsResult,
    compute_temp_top_p_normalized_logprobs,
    get_token_ids_logprobs_chunk,
    get_token_ids_logprobs_prefill,
    get_top_logprobs_chunk,
    get_top_logprobs_prefill,
)
from sglang.srt.layers.vocab_parallel_embedding import VocabParallelEmbedding
from sglang.srt.model_executor.forward_batch_info import (
    CaptureHiddenMode,
    ForwardBatch,
    ForwardMode,
)
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils.common import is_npu, use_intel_amx_backend

logger = logging.getLogger(__name__)

_is_npu = is_npu()
```
**EN:** This section prepares the module namespace. It imports `dataclasses`, `logging`, `typing.Any`, `typing.Dict`, `typing.List`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` and `_is_npu` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `dataclasses`、`logging`、`typing.Any`、`typing.Dict`、`typing.List` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 和 `_is_npu` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 66-110: Class `LogitsProcessorOutput` declaration and shared state
```python
@dataclasses.dataclass
class LogitsProcessorOutput:
    ## Part 1: This part will be assigned in python/sglang/srt/layers/logits_processor.py::LogitsProcessor
    # The logits of the next tokens.       shape: [#seq, vocab_size]
    # Can be None for certain prefill-only requests (e.g., multi-item scoring) that don't need next token generation
    next_token_logits: Optional[torch.Tensor]
    # Used by speculative decoding (EAGLE)
    # The last hidden layers
    hidden_states: Optional[torch.Tensor] = None

    ## Part 2: This part will be assigned in python/sglang/srt/layers/sampler.py::Sampler
    # he log probs of output tokens, if SGLANG_RETURN_ORIGINAL_LOGPROB = True, will get the log probs before applying temperature. If False, will get the log probs before applying temperature.
    next_token_logprobs: Optional[torch.Tensor] = None
    # The logprobs and ids of the top-k tokens in output positions. shape: [#seq, k]
    next_token_top_logprobs_val: Optional[List] = None
    next_token_top_logprobs_idx: Optional[List] = None
    # The logprobs and ids of the requested token ids in output positions. shape: [#seq, n] (n is the number of requested token ids)
    # Can contain either lists or GPU tensors (for delayed copy optimization in prefill-only requests)
    next_token_token_ids_logprobs_val: Optional[
        List[Union[List[float], torch.Tensor]]
    ] = None
    next_token_token_ids_logprobs_idx: Optional[List] = None

    ## Part 3: Prefill-only. This part will be assigned in python/sglang/srt/layers/logits_processor.py::LogitsProcessor
    # The logprobs of input tokens.        shape: [#token]
    input_token_logprobs: Optional[torch.Tensor] = None
    # The logprobs and ids of the top-k tokens in input positions.  shape: [#seq, #token, k]
    input_top_logprobs_val: Optional[List] = None
    input_top_logprobs_idx: Optional[List] = None
    # The logprobs and ids of the requested token ids in input positions. shape: [#seq, n] (n is the number of requested token ids)
    # Can contain either lists or GPU tensors (for delayed GPU-to-CPU transfer optimization)
    input_token_ids_logprobs_val: Optional[List[Union[List[float], torch.Tensor]]] = (
        None
    )
    input_token_ids_logprobs_idx: Optional[List] = None

    ## Part 4: Diffusion LLM only.
    full_logits: Optional[torch.Tensor] = None

    ## Part 5: Customized Info
    customized_info: Optional[Dict[str, List[Any]]] = None

    mm_input_embeds: Optional[torch.Tensor] = None
```
**EN:** This block introduces class `LogitsProcessorOutput` and the state shared by its methods.
**CN:** 该代码块引入类 `LogitsProcessorOutput`，并定义其方法共享的状态。

### Lines 111-153: Class `LogitsMetadata` declaration and shared state
```python
@dataclasses.dataclass
class LogitsMetadata:
    forward_mode: ForwardMode
    capture_hidden_mode: CaptureHiddenMode = CaptureHiddenMode.NULL
    next_token_logits_buffer: Optional[torch.Tensor] = None

    extend_return_logprob: bool = False
    extend_return_top_logprob: bool = False
    extend_token_ids_logprob: bool = False
    extend_seq_lens: Optional[torch.Tensor] = None
    extend_seq_lens_cpu: Optional[List[int]] = None
    extend_logprob_start_lens_cpu: Optional[List[int]] = None
    extend_logprob_pruned_lens_cpu: Optional[List[int]] = None
    top_logprobs_nums: Optional[List[int]] = None
    extend_input_logprob_token_ids_gpu: Optional[torch.Tensor] = None
    token_ids_logprobs: Optional[List[List[int]]] = None

    # logits and logprobs post processing
    temp_scaled_logprobs: bool = False
    temperature: torch.Tensor = None
    top_p_normalized_logprobs: bool = False
    top_p: torch.Tensor = None

    # DP attention metadata. Not needed when DP attention is not used.
    # Number of tokens in the request.
    global_num_tokens_gpu: Optional[torch.Tensor] = None
    # The start position of local hidden states.
    dp_local_start_pos: Optional[torch.Tensor] = None
    dp_local_num_tokens: Optional[torch.Tensor] = None
    global_dp_buffer_len: Optional[int] = None
    # Number of tokens to sample per DP rank
    global_num_tokens_for_logprob_cpu: Optional[torch.Tensor] = None
    global_num_tokens_for_logprob_gpu: Optional[torch.Tensor] = None
    # The gather mode for DP attention
    dp_padding_mode: Optional[DpPaddingMode] = None
    # for padding
    padded_static_len: int = -1

    # Whether this batch is prefill-only (no token generation needed)
    is_prefill_only: bool = False

    mm_input_embeds: Optional[torch.Tensor] = None
```
**EN:** This block introduces class `LogitsMetadata` and the state shared by its methods.
**CN:** 该代码块引入类 `LogitsMetadata`，并定义其方法共享的状态。

### Lines 154-206: Function `LogitsMetadata.from_forward_batch` and its core logic
```python
    @classmethod
    def from_forward_batch(cls, forward_batch: ForwardBatch):
        if (
            forward_batch.forward_mode.is_extend()
            and forward_batch.return_logprob
            and not forward_batch.forward_mode.is_target_verify()
        ):
            extend_return_top_logprob = any(
                x > 0 for x in forward_batch.top_logprobs_nums
            )
            extend_token_ids_logprob = any(
                x is not None for x in forward_batch.token_ids_logprobs
            )
            extend_return_logprob = False
            extend_logprob_pruned_lens_cpu = []
            for extend_len, start_len in zip(
                forward_batch.extend_seq_lens_cpu,
                forward_batch.extend_logprob_start_lens_cpu,
            ):
                if extend_len - start_len > 0:
                    extend_return_logprob = True
                extend_logprob_pruned_lens_cpu.append(extend_len - start_len)
        else:
            extend_return_logprob = extend_return_top_logprob = (
                extend_token_ids_logprob
            ) = extend_logprob_pruned_lens_cpu = False

        return cls(
            forward_mode=forward_batch.forward_mode,
            capture_hidden_mode=forward_batch.capture_hidden_mode,
            next_token_logits_buffer=forward_batch.next_token_logits_buffer,
            extend_return_logprob=extend_return_logprob,
            extend_return_top_logprob=extend_return_top_logprob,
            extend_token_ids_logprob=extend_token_ids_logprob,
            extend_seq_lens=forward_batch.extend_seq_lens,
            extend_seq_lens_cpu=forward_batch.extend_seq_lens_cpu,
            extend_logprob_start_lens_cpu=forward_batch.extend_logprob_start_lens_cpu,
            extend_logprob_pruned_lens_cpu=extend_logprob_pruned_lens_cpu,
            top_logprobs_nums=forward_batch.top_logprobs_nums,
            token_ids_logprobs=forward_batch.token_ids_logprobs,
            extend_input_logprob_token_ids_gpu=forward_batch.extend_input_logprob_token_ids_gpu,
            padded_static_len=forward_batch.padded_static_len,
            is_prefill_only=forward_batch.is_prefill_only,
            global_num_tokens_gpu=forward_batch.global_num_tokens_gpu,
            dp_local_start_pos=forward_batch.dp_local_start_pos,
            dp_local_num_tokens=forward_batch.dp_local_num_tokens,
            global_dp_buffer_len=forward_batch.global_dp_buffer_len,
            global_num_tokens_for_logprob_cpu=forward_batch.global_num_tokens_for_logprob_cpu,
            global_num_tokens_for_logprob_gpu=forward_batch.global_num_tokens_for_logprob_gpu,
            dp_padding_mode=DpPaddingMode.SUM_LEN,
            mm_input_embeds=forward_batch.mm_input_embeds,
        )
```
**EN:** This block defines `LogitsMetadata.from_forward_batch` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls`, `forward_batch.forward_mode.is_extend`, `any`, `zip`, and `forward_batch.forward_mode.is_target_verify`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `extend_return_top_logprob`, `extend_token_ids_logprob`, `extend_return_logprob`, and `extend_logprob_pruned_lens_cpu` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsMetadata.from_forward_batch`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls`、`forward_batch.forward_mode.is_extend`、`any`、`zip` 以及 `forward_batch.forward_mode.is_target_verify`，说明该流程会编排底层辅助函数或计算内核。 像 `extend_return_top_logprob`、`extend_token_ids_logprob`、`extend_return_logprob` 以及 `extend_logprob_pruned_lens_cpu` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 207-240: `LogitsMetadata.compute_dp_attention_metadata` step for data parallel attention metadata
```python
    def compute_dp_attention_metadata(self):

        cumtokens = torch.cumsum(self.global_num_tokens_for_logprob_gpu, dim=0)
        dp_rank = get_attention_dp_rank()
        if dp_rank == 0:
            dp_local_start_pos = torch.zeros_like(
                self.global_num_tokens_for_logprob_gpu[0]
            )
        else:
            dp_local_start_pos = cumtokens[dp_rank - 1]

        self.dp_local_start_pos = dp_local_start_pos
        self.dp_local_num_tokens = self.global_num_tokens_for_logprob_gpu[dp_rank]

        hidden_size = get_dp_hidden_size()
        dtype = get_dp_dtype()
        device = get_dp_device()

        if self.global_num_tokens_for_logprob_cpu is not None:
            # create a smaller buffer to reduce peak memory usage
            self.global_dp_buffer_len = sum(self.global_num_tokens_for_logprob_cpu)
        else:
            self.global_dp_buffer_len = self.global_dp_buffer_len

        self.gathered_buffer = torch.empty(
            (
                self.global_dp_buffer_len,
                hidden_size,
            ),
            dtype=dtype,
            device=device,
        )
```
**EN:** This block defines `LogitsMetadata.compute_dp_attention_metadata` and contains the main logic for this step. It mainly invokes `torch.cumsum`, `get_attention_dp_rank`, `get_dp_hidden_size`, `get_dp_dtype`, and `get_dp_device`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cumtokens`, `dp_rank`, `self.dp_local_start_pos`, `self.dp_local_num_tokens`, and `hidden_size` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LogitsMetadata.compute_dp_attention_metadata`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cumsum`、`get_attention_dp_rank`、`get_dp_hidden_size`、`get_dp_dtype` 以及 `get_dp_device`，说明该流程会编排底层辅助函数或计算内核。 像 `cumtokens`、`dp_rank`、`self.dp_local_start_pos`、`self.dp_local_num_tokens` 以及 `hidden_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 241-241: Class `LogitsProcessor` declaration and shared state
```python
class LogitsProcessor(nn.Module):
```
**EN:** This block introduces class `LogitsProcessor` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `LogitsProcessor`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。

### Lines 242-284: `LogitsProcessor` initialization and state setup
```python
    def __init__(
        self,
        config,
        skip_all_gather: bool = False,
        logit_scale: Optional[float] = None,
        return_full_logits: bool = False,
    ):
        super().__init__()
        self.config = config
        self.vocab_size = config.vocab_size
        self.logit_scale = logit_scale
        self.use_attn_tp_group = get_global_server_args().enable_dp_lm_head
        self.use_fp32_lm_head = get_global_server_args().enable_fp32_lm_head
        if self.use_attn_tp_group:
            self.attn_tp_size = get_attention_tp_size()
            self.do_tensor_parallel_all_gather = (
                not skip_all_gather and self.attn_tp_size > 1
            )
            self.do_tensor_parallel_all_gather_dp_attn = False
        else:
            self.do_tensor_parallel_all_gather = (
                not skip_all_gather and get_tensor_model_parallel_world_size() > 1
            )
            self.do_tensor_parallel_all_gather_dp_attn = (
                self.do_tensor_parallel_all_gather and get_attention_dp_size() != 1
            )
        self.final_logit_softcapping = getattr(
            self.config, "final_logit_softcapping", None
        )
        if (
            self.final_logit_softcapping is not None
            and self.final_logit_softcapping < 0
        ):
            self.final_logit_softcapping = None

        self.return_full_logits = return_full_logits
        self.enable_mis = get_global_server_args().enable_mis

        # enable chunked logprobs processing
        self.enable_logprobs_chunk = envs.SGLANG_ENABLE_LOGITS_PROCESSER_CHUNK.get()
        # chunk size for logprobs processing
        self.logprobs_chunk_size = envs.SGLANG_LOGITS_PROCESSER_CHUNK_SIZE.get()
```
**EN:** This block defines `LogitsProcessor.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `getattr`, `envs.SGLANG_ENABLE_LOGITS_PROCESSER_CHUNK.get`, `envs.SGLANG_LOGITS_PROCESSER_CHUNK_SIZE.get`, and `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.config`, `self.vocab_size`, `self.logit_scale`, `self.use_attn_tp_group`, and `self.use_fp32_lm_head` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LogitsProcessor.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`getattr`、`envs.SGLANG_ENABLE_LOGITS_PROCESSER_CHUNK.get`、`envs.SGLANG_LOGITS_PROCESSER_CHUNK_SIZE.get` 以及 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 像 `self.config`、`self.vocab_size`、`self.logit_scale`、`self.use_attn_tp_group` 以及 `self.use_fp32_lm_head` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 285-403: `LogitsProcessor.forward` main forward path
```python
    def forward(
        self,
        input_ids,
        hidden_states,
        lm_head: VocabParallelEmbedding,
        logits_metadata: Union[LogitsMetadata, ForwardBatch],
        aux_hidden_states: Optional[torch.Tensor] = None,
        hidden_states_before_norm: Optional[torch.Tensor] = None,
    ) -> LogitsProcessorOutput:
        # Extract MIS indices before ForwardBatch → LogitsMetadata conversion
        multi_item_delimiter_indices = None
        if isinstance(logits_metadata, ForwardBatch):
            multi_item_delimiter_indices = logits_metadata.multi_item_delimiter_indices
            logits_metadata = LogitsMetadata.from_forward_batch(logits_metadata)

        # Multi-item scoring only for prefill-only requests with pre-computed indices.
        if multi_item_delimiter_indices is not None and logits_metadata.is_prefill_only:
            return self.compute_logprobs_for_multi_item_scoring(
                input_ids,
                hidden_states,
                lm_head,
                logits_metadata,
                multi_item_delimiter_indices,
            )

        # Diffusion LLM only.
        if logits_metadata.forward_mode.is_dllm_extend():
            return self._get_dllm_logits(hidden_states, lm_head, logits_metadata)

        # Get the last hidden states and last logits for the next token prediction
        (
            pruned_states,
            pruned_states_before_norm,
            aux_pruned_states,
            sample_indices,
            input_logprob_indices,
            token_to_seq_idx,
        ) = self._get_pruned_states(
            hidden_states,
            hidden_states_before_norm,
            aux_hidden_states,
            logits_metadata,
        )

        hidden_states_to_store = self._get_hidden_states_to_store(
            hidden_states,
            hidden_states_before_norm,
            aux_hidden_states,
            pruned_states,
            pruned_states_before_norm,
            aux_pruned_states,
            sample_indices,
            logits_metadata,
        )
        del hidden_states

        if not logits_metadata.extend_return_logprob:
            # Compute logits for both input and sampled tokens.
            logits = self._get_logits(pruned_states, lm_head, logits_metadata)
            sampled_logits = (
                logits[sample_indices] if sample_indices is not None else logits
            )

            # Decode mode or extend mode without return_logprob.
            return LogitsProcessorOutput(
                next_token_logits=sampled_logits,
                hidden_states=hidden_states_to_store,
                # FIXME: These fields are not logits-related but are passed through here as a
                # workaround since ForwardBatch is local to forward_batch_generation().
                # They should be moved to GenerationBatchResult to keep this class clean.
                mm_input_embeds=logits_metadata.mm_input_embeds,
            )

        # Start to process input logprobs
        # Normalize the logprob w/o temperature, top-p
        self._expand_metadata_for_logprobs(logits_metadata, pruned_states.device)

        # Determine whether to use chunked or non-chunked logits processing.
        # Skip chunking if:
        # 1. Chunking is disabled
        # 2. Total count is below chunk size threshold
        # 3. DP attention all-gather is enabled (can use "enable_dp_lm_head" to enable chunking)
        should_skip_chunking = (
            not self.enable_logprobs_chunk
            or pruned_states.shape[0] <= self.logprobs_chunk_size
            or self.do_tensor_parallel_all_gather_dp_attn
        )

        if should_skip_chunking:
            # Compute logits for both input and sampled tokens.
            logits = self._get_logits(pruned_states, lm_head, logits_metadata)
            sampled_logits = (
                logits[sample_indices] if sample_indices is not None else logits
            )
            input_logits = logits[input_logprob_indices]
            del logits

            logprobs_result = self.process_input_logprobs(input_logits, logits_metadata)
        else:
            logprobs_result, sampled_logits = self.process_input_logprobs_by_chunk(
                pruned_states,
                sample_indices,
                input_logprob_indices,
                token_to_seq_idx,
                lm_head,
                logits_metadata,
            )

        return LogitsProcessorOutput(
            next_token_logits=sampled_logits,
            hidden_states=hidden_states_to_store,
            input_token_logprobs=logprobs_result.input_token_logprobs,
            input_top_logprobs_val=logprobs_result.input_top_logprobs_val,
            input_top_logprobs_idx=logprobs_result.input_top_logprobs_idx,
            input_token_ids_logprobs_val=logprobs_result.input_token_ids_logprobs_val,
            input_token_ids_logprobs_idx=logprobs_result.input_token_ids_logprobs_idx,
            mm_input_embeds=logits_metadata.mm_input_embeds,
        )
```
**EN:** This block defines `LogitsProcessor.forward` and contains the main logic for this step. It mainly invokes `isinstance`, `logits_metadata.forward_mode.is_dllm_extend`, `self._get_pruned_states`, `self._get_hidden_states_to_store`, and `self._expand_metadata_for_logprobs`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `multi_item_delimiter_indices`, `pruned_states`, `pruned_states_before_norm`, `aux_pruned_states`, and `sample_indices` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor.forward`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance`、`logits_metadata.forward_mode.is_dllm_extend`、`self._get_pruned_states`、`self._get_hidden_states_to_store` 以及 `self._expand_metadata_for_logprobs`，说明该流程会编排底层辅助函数或计算内核。 像 `multi_item_delimiter_indices`、`pruned_states`、`pruned_states_before_norm`、`aux_pruned_states` 以及 `sample_indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 404-556: Internal helper `LogitsProcessor._get_pruned_states`
```python
    def _get_pruned_states(
        self,
        hidden_states: torch.Tensor,
        hidden_states_before_norm: Optional[torch.Tensor],
        aux_hidden_states: Optional[torch.Tensor],
        logits_metadata: LogitsMetadata,
    ):
        pruned_states_before_norm: Optional[torch.Tensor] = None
        aux_pruned_states = None
        token_to_seq_idx = []

        if (
            logits_metadata.forward_mode.is_decode_or_idle()
            or logits_metadata.forward_mode.is_target_verify()
            or logits_metadata.forward_mode.is_draft_extend_v2()
        ):
            pruned_states = hidden_states
            pruned_states_before_norm = hidden_states_before_norm
            if aux_hidden_states is not None:
                aux_pruned_states = [hidden for hidden in aux_hidden_states]
            sample_indices = None
            input_logprob_indices = None

        elif (
            logits_metadata.forward_mode.is_extend()
            and not logits_metadata.extend_return_logprob
        ):
            # Prefill without input logprobs.
            if logits_metadata.padded_static_len < 0:
                last_index = torch.cumsum(logits_metadata.extend_seq_lens, dim=0) - 1
            else:
                # If padding_static length is 5 and extended_seq_lens is [2, 3],
                # then our batch looks like [t00, t01, p, p, p, t10, t11, t12, p, p]
                # and this retrieves t01 and t12, which are the valid last tokens
                idx = torch.arange(
                    len(logits_metadata.extend_seq_lens),
                    device=logits_metadata.extend_seq_lens.device,
                )
                last_index = (
                    idx * logits_metadata.padded_static_len
                    + logits_metadata.extend_seq_lens
                    - 1
                )
            pruned_states = hidden_states[last_index]
            if hidden_states_before_norm is not None:
                pruned_states_before_norm = hidden_states_before_norm[last_index]
            if aux_hidden_states is not None:
                aux_pruned_states = [hidden[last_index] for hidden in aux_hidden_states]
            sample_indices = None
            input_logprob_indices = None
        else:
            # Prefill with input logprobs.
            # Find 4 different indices.
            # 1. pruned_states: hidden states that we want logprobs from.
            # 2. sample_indices: Indices that have sampled tokens.
            # 3. input_logprob_indices: Indices that have input logprob tokens.
            # 4. token_to_seq_idx: map each token to its sequence index
            #
            # Example
            # -------
            # Suppose a batch (flattened by sequence):
            # [t00, t01, t02, t03, t10, t11, t12, t13, t14, t20, t21, t22, t23, t24, t25]
            # extend_seq_lens_cpu           = [4, 5, 6]
            # extend_logprob_start_lens_cpu = [0, 5, 3]
            #
            # Then, the indices are:
            # pruned_states         -> [t00, t01, t02, t03, t14, t23, t24, t25]
            # sample_indices        -> [3, 4, 7]
            # input_logprob_indices -> [0, 1, 2, 3, 5, 6, 7]
            # token_to_seq_idx      -> [0, 0, 0, 0, 1, 2, 2, 2]
            #
            # If chunk is enabled and chunk_size = 3, the chunks will be computed in a chunked manner:
            # [t00, t01, t02], [t03, t14, t23], [t24, t25]

            sample_index_pt = -1
            sample_indices = []
            input_logprob_indices_pt = 0
            input_logprob_indices = []
            pt, pruned_states_list, pruned_states_before_norm_list = 0, [], []
            aux_pruned_states_lists = (
                [[] for _ in aux_hidden_states]
                if aux_hidden_states is not None
                else None
            )

            for idx, (extend_logprob_start_len, extend_len) in enumerate(
                zip(
                    logits_metadata.extend_logprob_start_lens_cpu,
                    logits_metadata.extend_seq_lens_cpu,
                )
            ):
                # It can happen in chunked prefill. We still need to sample 1 token,
                # But we don't want to include it in input logprob.
                if extend_len == extend_logprob_start_len:
                    start_len = extend_logprob_start_len - 1
                else:
                    start_len = extend_logprob_start_len

                # We always need at least 1 token to sample because that's required
                # by a caller.
                assert extend_len > start_len
                pruned_states_list.append(
                    hidden_states[pt + start_len : pt + extend_len]
                )
                if hidden_states_before_norm is not None:
                    pruned_states_before_norm_list.append(
                        hidden_states_before_norm[pt + start_len : pt + extend_len]
                    )
                if aux_pruned_states_lists is not None:
                    for j, hidden in enumerate(aux_hidden_states):
                        aux_pruned_states_lists[j].append(
                            hidden[pt + start_len : pt + extend_len]
                        )
                # Map each token to its sequence index, for chunked computation
                # of input logprobs
                token_to_seq_idx.extend([idx] * (extend_len - start_len))
                pt += extend_len
                sample_index_pt += extend_len - start_len
                sample_indices.append(sample_index_pt)
                input_logprob_indices.extend(
                    [
                        input_logprob_indices_pt + i
                        for i in range(extend_len - extend_logprob_start_len)
                    ]
                )
                input_logprob_indices_pt += extend_len - start_len

            # Set the last token of the last sequence
            token_to_seq_idx.append(len(logits_metadata.extend_seq_lens_cpu) - 1)
            pruned_states = torch.cat(pruned_states_list)
            if hidden_states_before_norm is not None:
                pruned_states_before_norm = torch.cat(pruned_states_before_norm_list)
            if aux_pruned_states_lists is not None:
                aux_pruned_states = [torch.cat(lst) for lst in aux_pruned_states_lists]

            # Build the index tensors via pinned host memory + non-blocking H2D
            # so the small copy doesn't drain the stream.
            sample_indices = torch.tensor(
                sample_indices, dtype=torch.int64, pin_memory=True
            ).to(pruned_states.device, non_blocking=True)
            input_logprob_indices = torch.tensor(
                input_logprob_indices, dtype=torch.int64, pin_memory=True
            ).to(pruned_states.device, non_blocking=True)

        return (
            pruned_states,
            pruned_states_before_norm,
            aux_pruned_states,
            sample_indices,
            input_logprob_indices,
            token_to_seq_idx,
        )
```
**EN:** This block defines `LogitsProcessor._get_pruned_states` and contains the main logic for this step. It mainly invokes `logits_metadata.forward_mode.is_decode_or_idle`, `logits_metadata.forward_mode.is_target_verify`, `logits_metadata.forward_mode.is_draft_extend_v2`, `logits_metadata.forward_mode.is_extend`, and `enumerate`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pruned_states_before_norm`, `aux_pruned_states`, `token_to_seq_idx`, `pruned_states`, and `sample_indices` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._get_pruned_states`，并承载这一阶段的核心逻辑。 它主要调用 `logits_metadata.forward_mode.is_decode_or_idle`、`logits_metadata.forward_mode.is_target_verify`、`logits_metadata.forward_mode.is_draft_extend_v2`、`logits_metadata.forward_mode.is_extend` 以及 `enumerate`，说明该流程会编排底层辅助函数或计算内核。 像 `pruned_states_before_norm`、`aux_pruned_states`、`token_to_seq_idx`、`pruned_states` 以及 `sample_indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 557-609: Internal helper `LogitsProcessor._get_hidden_states_to_store`
```python
    def _get_hidden_states_to_store(
        self,
        hidden_states: torch.Tensor,
        hidden_states_before_norm: Optional[torch.Tensor],
        aux_hidden_states: Optional[List[torch.Tensor]],
        pruned_states: torch.Tensor,
        pruned_states_before_norm: Optional[torch.Tensor],
        aux_pruned_states: Optional[List[torch.Tensor]],
        sample_indices: Optional[torch.Tensor],
        logits_metadata: LogitsMetadata,
    ) -> Optional[torch.Tensor]:
        hidden_states_to_store: Optional[torch.Tensor] = None
        hidden_states_to_store_before_norm: Optional[torch.Tensor] = None
        if logits_metadata.capture_hidden_mode.need_capture():
            if logits_metadata.capture_hidden_mode.is_full():
                if aux_hidden_states is not None:
                    aux_hidden_states = torch.cat(aux_hidden_states, dim=-1)
                    hidden_states_to_store = aux_hidden_states
                else:
                    hidden_states_to_store = hidden_states
                hidden_states_to_store_before_norm = hidden_states_before_norm
            elif logits_metadata.capture_hidden_mode.is_last():
                # Get the last token hidden states. If sample_indices is None,
                # pruned states only contain the last tokens already.
                if aux_hidden_states is not None:
                    aux_pruned_states = torch.cat(aux_pruned_states, dim=-1)
                    hidden_states_to_store = (
                        aux_pruned_states[sample_indices]
                        if sample_indices is not None
                        else aux_pruned_states
                    )
                else:
                    hidden_states_to_store = (
                        pruned_states[sample_indices]
                        if sample_indices is not None
                        else pruned_states
                    )
                    if hidden_states_before_norm is not None:
                        hidden_states_to_store_before_norm = (
                            pruned_states_before_norm[sample_indices]
                            if sample_indices is not None
                            else pruned_states_before_norm
                        )
            else:
                assert False, "Should never reach"

        if hidden_states_to_store_before_norm is not None:
            # NOTE: when hidden_states_before_norm is provided, we always
            # prefer to return it.
            hidden_states_to_store = hidden_states_to_store_before_norm

        return hidden_states_to_store
```
**EN:** This block defines `LogitsProcessor._get_hidden_states_to_store` and contains the main logic for this step. It mainly invokes `logits_metadata.capture_hidden_mode.need_capture`, `logits_metadata.capture_hidden_mode.is_full`, `logits_metadata.capture_hidden_mode.is_last`, and `torch.cat`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states_to_store`, `hidden_states_to_store_before_norm`, `aux_hidden_states`, and `aux_pruned_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._get_hidden_states_to_store`，并承载这一阶段的核心逻辑。 它主要调用 `logits_metadata.capture_hidden_mode.need_capture`、`logits_metadata.capture_hidden_mode.is_full`、`logits_metadata.capture_hidden_mode.is_last` 以及 `torch.cat`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states_to_store`、`hidden_states_to_store_before_norm`、`aux_hidden_states` 以及 `aux_pruned_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 610-628: Internal helper `LogitsProcessor._expand_metadata_for_logprobs`
```python
    def _expand_metadata_for_logprobs(
        self, logits_metadata: LogitsMetadata, device: torch.device
    ):
        pruned_lens = torch.tensor(
            logits_metadata.extend_logprob_pruned_lens_cpu,
            dtype=torch.int64,
            pin_memory=True,
        ).to(device, non_blocking=True)
        if logits_metadata.temp_scaled_logprobs:
            logits_metadata.temperature = torch.repeat_interleave(
                logits_metadata.temperature.view(-1),
                pruned_lens,
            ).view(-1, 1)
        if logits_metadata.top_p_normalized_logprobs:
            logits_metadata.top_p = torch.repeat_interleave(
                logits_metadata.top_p,
                pruned_lens,
            )
```
**EN:** This block defines `LogitsProcessor._expand_metadata_for_logprobs` and contains the main logic for this step. It mainly invokes `torch.tensor.to`, `torch.repeat_interleave.view`, `torch.repeat_interleave`, `torch.tensor`, and `logits_metadata.temperature.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pruned_lens` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LogitsProcessor._expand_metadata_for_logprobs`，并承载这一阶段的核心逻辑。 它主要调用 `torch.tensor.to`、`torch.repeat_interleave.view`、`torch.repeat_interleave`、`torch.tensor` 以及 `logits_metadata.temperature.view`，说明该流程会编排底层辅助函数或计算内核。 像 `pruned_lens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 629-664: Function `LogitsProcessor.process_input_logprobs` and its core logic
```python
    def process_input_logprobs(self, input_logits, logits_metadata: LogitsMetadata):
        input_logprobs = compute_temp_top_p_normalized_logprobs(
            input_logits, logits_metadata
        )

        # Get the logprob of top-k tokens
        if logits_metadata.extend_return_top_logprob:
            (
                input_top_logprobs_val,
                input_top_logprobs_idx,
            ) = get_top_logprobs_prefill(input_logprobs, logits_metadata)
        else:
            input_top_logprobs_val = input_top_logprobs_idx = None

        # Get the logprob of given token id
        if logits_metadata.extend_token_ids_logprob:
            (
                input_token_ids_logprobs_val,
                input_token_ids_logprobs_idx,
            ) = get_token_ids_logprobs_prefill(input_logprobs, logits_metadata)
        else:
            input_token_ids_logprobs_val = input_token_ids_logprobs_idx = None

        input_token_logprobs = input_logprobs[
            torch.arange(input_logprobs.shape[0], device=input_logprobs.device),
            logits_metadata.extend_input_logprob_token_ids_gpu,
        ]

        return InputLogprobsResult(
            input_token_logprobs=input_token_logprobs,
            input_top_logprobs_val=input_top_logprobs_val,
            input_top_logprobs_idx=input_top_logprobs_idx,
            input_token_ids_logprobs_val=input_token_ids_logprobs_val,
            input_token_ids_logprobs_idx=input_token_ids_logprobs_idx,
        )
```
**EN:** This block defines `LogitsProcessor.process_input_logprobs` and contains the main logic for this step. It mainly invokes `compute_temp_top_p_normalized_logprobs`, `InputLogprobsResult`, `get_top_logprobs_prefill`, `get_token_ids_logprobs_prefill`, and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `input_logprobs`, `input_token_logprobs`, `input_top_logprobs_val`, `input_top_logprobs_idx`, and `input_token_ids_logprobs_val` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor.process_input_logprobs`，并承载这一阶段的核心逻辑。 它主要调用 `compute_temp_top_p_normalized_logprobs`、`InputLogprobsResult`、`get_top_logprobs_prefill`、`get_token_ids_logprobs_prefill` 以及 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `input_logprobs`、`input_token_logprobs`、`input_top_logprobs_val`、`input_top_logprobs_idx` 以及 `input_token_ids_logprobs_val` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 665-842: Function `LogitsProcessor.process_input_logprobs_by_chunk` and its core logic
```python
    def process_input_logprobs_by_chunk(
        self,
        pruned_states: torch.Tensor,
        sample_indices: torch.Tensor,
        input_logprob_indices: torch.Tensor,
        token_to_seq_idx: list[int],
        lm_head: VocabParallelEmbedding,
        logits_metadata: LogitsMetadata,
    ) -> Tuple[InputLogprobsResult, torch.Tensor]:
        """
        compute logprobs for the output token from the hidden states.
        To avoid using too much memory, we split pruned_states into chunks of
        rows to compute input_logprobs separately, then concatenate the results.

        Returns:
            InputLogprobsResult: logprobs result
            torch.Tensor: sampled logits
        """

        # The peak memory usage is proportional to the chunk size.
        chunk_size = self.logprobs_chunk_size
        total_size = pruned_states.shape[0]
        num_chunks = (total_size + chunk_size - 1) // chunk_size

        input_token_logprobs = []
        if logits_metadata.extend_return_top_logprob:
            input_top_logprobs_val = []
            input_top_logprobs_idx = []
        else:
            input_top_logprobs_val = None
            input_top_logprobs_idx = None
        if logits_metadata.extend_token_ids_logprob:
            input_token_ids_logprobs_val = []
            input_token_ids_logprobs_idx = []
        else:
            input_token_ids_logprobs_val = None
            input_token_ids_logprobs_idx = None

        # If a single sequence is split into multiple chunks, we need to keep track
        # of the pruned length of the sequences in the previous chunks.
        split_len_topk = 0
        split_len_token_ids = 0

        for i in range(num_chunks):
            start_idx = i * chunk_size
            end_idx = min((i + 1) * chunk_size, total_size)

            # Notify lm_head LoRA about the current chunk so it can swap
            # to the precomputed per-chunk batch_info.  This is a no-op
            # for non-LoRA lm_head modules.
            if hasattr(lm_head, "set_lm_head_pass"):
                lm_head.set_lm_head_pass(i)

            # Get indices for this chunk
            chunk_mask = (input_logprob_indices >= start_idx) & (
                input_logprob_indices < end_idx
            )
            global_indices = input_logprob_indices[chunk_mask]
            chunk_indices = global_indices - start_idx
            # Get the positions in the original array where chunk_mask is True
            # This is needed to correctly index into extend_input_logprob_token_ids_gpu
            mask_indices = torch.nonzero(chunk_mask, as_tuple=True)[0]

            # Get the logits for this chunk
            chunk_states = pruned_states[start_idx:end_idx]
            chunk_logits = self._get_logits(chunk_states, lm_head, logits_metadata)

            # Initialize sampled_logits on first chunk
            if i == 0:
                sampled_logits = torch.empty(
                    (sample_indices.shape[0], chunk_logits.shape[1]),
                    dtype=chunk_logits.dtype,
                    device=chunk_logits.device,
                )

            # Handle sampled logits for the chunk if needed
            # This must be done before the continue statement to ensure all sampled_logits are filled
            chunk_sample_mask = (sample_indices >= start_idx) & (
                sample_indices < end_idx
            )
            if chunk_sample_mask.any():
                chunk_sample_indices = sample_indices[chunk_sample_mask] - start_idx
                sampled_logits[chunk_sample_mask] = chunk_logits[chunk_sample_indices]

            # If there are no input logprobs in this chunk, skip the rest
            if chunk_indices.numel() == 0:
                continue

            # Compute the logprobs of the chunk
            chunk_input_logprobs = chunk_logits[chunk_indices]
            # Only index per-token arrays when the corresponding feature is active.
            # Otherwise these tensors can be per-sequence (or scalars), which can
            # cause out-of-bounds indexing on GPU.
            chunk_temperature = (
                logits_metadata.temperature[global_indices]
                if logits_metadata.temp_scaled_logprobs
                and logits_metadata.temperature is not None
                else None
            )
            chunk_top_p = (
                logits_metadata.top_p[global_indices]
                if logits_metadata.top_p_normalized_logprobs
                and logits_metadata.top_p is not None
                else None
            )
            chunk_input_logprobs = compute_temp_top_p_normalized_logprobs(
                chunk_input_logprobs,
                logits_metadata,
                chunk_top_p,
                chunk_temperature,
            )

            # For each chunk, we need to get the slice of the token_to_seq_idx
            chunk_slice = slice(
                token_to_seq_idx[start_idx], token_to_seq_idx[end_idx] + 1
            )

            # Get the logprob of top-k tokens
            if logits_metadata.extend_return_top_logprob:
                top_k_nums = logits_metadata.top_logprobs_nums[chunk_slice]
                pruned_lens = logits_metadata.extend_logprob_pruned_lens_cpu[
                    chunk_slice
                ]
                split_len_topk = get_top_logprobs_chunk(
                    chunk_input_logprobs,
                    logits_metadata,
                    top_k_nums,
                    pruned_lens,
                    input_top_logprobs_val,
                    input_top_logprobs_idx,
                    split_len_topk,
                )

            # Get the logprob of given token id
            if logits_metadata.extend_token_ids_logprob:
                token_ids_logprobs = logits_metadata.token_ids_logprobs[chunk_slice]
                pruned_lens = logits_metadata.extend_logprob_pruned_lens_cpu[
                    chunk_slice
                ]
                split_len_token_ids = get_token_ids_logprobs_chunk(
                    chunk_input_logprobs,
                    token_ids_logprobs,
                    pruned_lens,
                    input_token_ids_logprobs_val,
                    input_token_ids_logprobs_idx,
                    split_len_token_ids,
                )

            # Get the logprob of the requested token ids
            chunk_input_token_logprobs = chunk_input_logprobs[
                torch.arange(
                    chunk_input_logprobs.shape[0], device=chunk_input_logprobs.device
                ),
                logits_metadata.extend_input_logprob_token_ids_gpu[mask_indices],
            ]
            input_token_logprobs.append(chunk_input_token_logprobs)

        # Restore the full-pruned lm_head batch_info after chunk iteration.
        if hasattr(lm_head, "reset_lm_head_pass"):
            assert hasattr(
                lm_head, "set_lm_head_pass"
            ), "lm_head must have set_lm_head_pass method and reset_lm_head_pass method at the same time"
            lm_head.reset_lm_head_pass()

        # Concatenate the results
        input_token_logprobs = torch.cat(input_token_logprobs, dim=0)

        return (
            InputLogprobsResult(
                input_token_logprobs=input_token_logprobs,
                input_top_logprobs_val=input_top_logprobs_val,
                input_top_logprobs_idx=input_top_logprobs_idx,
                input_token_ids_logprobs_val=input_token_ids_logprobs_val,
                input_token_ids_logprobs_idx=input_token_ids_logprobs_idx,
            ),
            sampled_logits,
        )
```
**EN:** This block defines `LogitsProcessor.process_input_logprobs_by_chunk` and contains the main logic for this step. It mainly invokes `range`, `hasattr`, `torch.cat`, `min`, and `self._get_logits`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `chunk_size`, `total_size`, `num_chunks`, `input_token_logprobs`, and `split_len_topk` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor.process_input_logprobs_by_chunk`，并承载这一阶段的核心逻辑。 它主要调用 `range`、`hasattr`、`torch.cat`、`min` 以及 `self._get_logits`，说明该流程会编排底层辅助函数或计算内核。 像 `chunk_size`、`total_size`、`num_chunks`、`input_token_logprobs` 以及 `split_len_topk` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 843-886: Internal helper `LogitsProcessor._get_logits`
```python
    def _get_logits(
        self,
        hidden_states: torch.Tensor,
        lm_head: VocabParallelEmbedding,
        logits_metadata: LogitsMetadata,
        embedding_bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        """Get logits from hidden_states.

        If sampled_logits_only is True, it means hidden_states only contain the
        last position (e.g., extend without input logprobs). The caller should
        guarantee the given hidden_states follow this constraint.
        """
        hidden_states, local_hidden_states = self._gather_dp_attn_hidden_states(
            hidden_states, logits_metadata
        )

        logits = self._compute_lm_head(hidden_states, lm_head, embedding_bias)

        if self.logit_scale is not None:
            logits.mul_(self.logit_scale)

        if self.do_tensor_parallel_all_gather:
            if self.use_attn_tp_group:
                logits = self._gather_attn_tp_logits(logits)
            else:
                logits = tensor_model_parallel_all_gather(logits)

        logits = self._scatter_dp_attn_logits(
            logits, local_hidden_states, logits_metadata
        )

        logits = self._copy_logits_to_buffer(logits, logits_metadata)

        if self.final_logit_softcapping:
            if not _is_npu:
                fused_softcap(logits, self.final_logit_softcapping)
            else:
                logits = self.final_logit_softcapping * torch.tanh(
                    logits / self.final_logit_softcapping
                )

        return logits
```
**EN:** This block defines `LogitsProcessor._get_logits` and contains the main logic for this step. It mainly invokes `self._gather_dp_attn_hidden_states`, `self._compute_lm_head`, `self._scatter_dp_attn_logits`, `self._copy_logits_to_buffer`, and `logits.mul_`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `local_hidden_states`, and `logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._get_logits`，并承载这一阶段的核心逻辑。 它主要调用 `self._gather_dp_attn_hidden_states`、`self._compute_lm_head`、`self._scatter_dp_attn_logits`、`self._copy_logits_to_buffer` 以及 `logits.mul_`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`local_hidden_states` 以及 `logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 887-931: Internal helper `LogitsProcessor._compute_lm_head`
```python
    def _compute_lm_head(
        self,
        hidden_states: torch.Tensor,
        lm_head: VocabParallelEmbedding,
        embedding_bias: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if hasattr(lm_head, "set_lora") and hasattr(lm_head, "apply_lora"):
            # This is a LoRA-wrapped module, use its forward method
            logits = lm_head(hidden_states)
        elif hasattr(lm_head, "weight"):
            # Normal linear layer
            if self.use_fp32_lm_head:
                logits = torch.matmul(
                    hidden_states.to(torch.float32), lm_head.weight.to(torch.float32).T
                )
            elif use_intel_amx_backend(lm_head):
                logits = torch.ops.sgl_kernel.weight_packed_linear(
                    hidden_states.to(lm_head.weight.dtype),
                    lm_head.weight,
                    None,  # bias
                    True,  # is_vnni
                )
            elif get_global_server_args().rl_on_policy_target is not None:
                # Due to tie-weight, we may not be able to change lm_head's weight dtype
                logits = torch.matmul(
                    hidden_states.bfloat16(), lm_head.weight.T.bfloat16()
                )
            else:
                logits = torch.matmul(
                    hidden_states.to(lm_head.weight.dtype), lm_head.weight.T
                )
        else:
            # GGUF models
            # TODO: use weight_packed_linear for GGUF models
            if self.use_fp32_lm_head:
                with torch.cuda.amp.autocast(enabled=False):
                    logits = lm_head.quant_method.apply(
                        lm_head, hidden_states.to(torch.float32), embedding_bias
                    )
            else:
                logits = lm_head.quant_method.apply(
                    lm_head, hidden_states, embedding_bias
                )
        return logits
```
**EN:** This block defines `LogitsProcessor._compute_lm_head` and contains the main logic for this step. It mainly invokes `hasattr`, `lm_head`, `torch.matmul`, `use_intel_amx_backend`, and `lm_head.quant_method.apply`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._compute_lm_head`，并承载这一阶段的核心逻辑。 它主要调用 `hasattr`、`lm_head`、`torch.matmul`、`use_intel_amx_backend` 以及 `lm_head.quant_method.apply`，说明该流程会编排底层辅助函数或计算内核。 像 `logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 932-942: Internal helper `LogitsProcessor._gather_dp_attn_hidden_states`
```python
    def _gather_dp_attn_hidden_states(
        self, hidden_states: torch.Tensor, logits_metadata: LogitsMetadata
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if self.do_tensor_parallel_all_gather_dp_attn:
            logits_metadata.compute_dp_attention_metadata()
            local_hidden_states = hidden_states
            hidden_states = logits_metadata.gathered_buffer
            dp_gather_replicate(hidden_states, local_hidden_states, logits_metadata)
            return hidden_states, local_hidden_states
        return hidden_states, hidden_states
```
**EN:** This block defines `LogitsProcessor._gather_dp_attn_hidden_states` and contains the main logic for this step. It mainly invokes `logits_metadata.compute_dp_attention_metadata` and `dp_gather_replicate`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `local_hidden_states` and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._gather_dp_attn_hidden_states`，并承载这一阶段的核心逻辑。 它主要调用 `logits_metadata.compute_dp_attention_metadata` 和 `dp_gather_replicate`，说明该流程会编排底层辅助函数或计算内核。 像 `local_hidden_states` 和 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 943-970: Internal helper `LogitsProcessor._gather_attn_tp_logits`
```python
    def _gather_attn_tp_logits(self, logits: torch.Tensor) -> torch.Tensor:
        if self.vocab_size % self.attn_tp_size == 0:
            global_logits = torch.empty(
                (
                    self.attn_tp_size,
                    logits.shape[0],
                    self.vocab_size // self.attn_tp_size,
                ),
                device=logits.device,
                dtype=logits.dtype,
            )
            attn_tp_all_gather_into_tensor(global_logits, logits)
            global_logits = global_logits.permute(1, 0, 2).reshape(
                logits.shape[0], self.vocab_size
            )
        else:
            global_logits = torch.empty(
                (self.vocab_size, logits.shape[0]),
                device=logits.device,
                dtype=logits.dtype,
            )
            global_logits = global_logits.T
            attn_tp_all_gather(
                list(global_logits.tensor_split(self.attn_tp_size, dim=-1)),
                logits,
            )
        return global_logits
```
**EN:** This block defines `LogitsProcessor._gather_attn_tp_logits` and contains the main logic for this step. It mainly invokes `torch.empty`, `attn_tp_all_gather_into_tensor`, `global_logits.permute.reshape`, `attn_tp_all_gather`, and `list`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `global_logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._gather_attn_tp_logits`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`attn_tp_all_gather_into_tensor`、`global_logits.permute.reshape`、`attn_tp_all_gather` 以及 `list`，说明该流程会编排底层辅助函数或计算内核。 像 `global_logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 971-986: Internal helper `LogitsProcessor._scatter_dp_attn_logits`
```python
    def _scatter_dp_attn_logits(
        self,
        logits: torch.Tensor,
        local_hidden_states: torch.Tensor,
        logits_metadata: LogitsMetadata,
    ) -> torch.Tensor:
        if self.do_tensor_parallel_all_gather_dp_attn:
            global_logits = logits
            logits = torch.empty(
                (local_hidden_states.shape[0], global_logits.shape[1]),
                device=global_logits.device,
                dtype=global_logits.dtype,
            )
            dp_scatter(logits, global_logits, logits_metadata)
        return logits
```
**EN:** This block defines `LogitsProcessor._scatter_dp_attn_logits` and contains the main logic for this step. It mainly invokes `torch.empty` and `dp_scatter`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `global_logits` and `logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._scatter_dp_attn_logits`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `dp_scatter`，说明该流程会编排底层辅助函数或计算内核。 像 `global_logits` 和 `logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 987-998: Internal helper `LogitsProcessor._copy_logits_to_buffer`
```python
    def _copy_logits_to_buffer(
        self, logits: torch.Tensor, logits_metadata: LogitsMetadata
    ) -> torch.Tensor:
        if logits_metadata.next_token_logits_buffer is not None:
            logits_buffer = logits_metadata.next_token_logits_buffer
            assert logits_buffer.dtype == torch.float
            logits_buffer.copy_(logits[:, : self.vocab_size])
            logits = logits_buffer
        else:
            logits = logits[:, : self.vocab_size].float()
        return logits
```
**EN:** This block defines `LogitsProcessor._copy_logits_to_buffer` and contains the main logic for this step. It mainly invokes `logits_buffer.copy_` and `logits.float`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `logits_buffer` and `logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._copy_logits_to_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `logits_buffer.copy_` 和 `logits.float`，说明该流程会编排底层辅助函数或计算内核。 像 `logits_buffer` 和 `logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 999-1011: Internal helper `LogitsProcessor._get_dllm_logits`
```python
    def _get_dllm_logits(
        self,
        hidden_states: torch.Tensor,
        lm_head: VocabParallelEmbedding,
        logits_metadata: LogitsMetadata,
    ) -> LogitsProcessorOutput:
        assert self.return_full_logits
        full_logits = self._get_logits(hidden_states, lm_head, logits_metadata)
        return LogitsProcessorOutput(
            full_logits=full_logits,
            next_token_logits=None,
        )
```
**EN:** This block defines `LogitsProcessor._get_dllm_logits` and contains the main logic for this step. It mainly invokes `self._get_logits` and `LogitsProcessorOutput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `full_logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor._get_dllm_logits`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_logits` 和 `LogitsProcessorOutput`，说明该流程会编排底层辅助函数或计算内核。 像 `full_logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1012-1112: `LogitsProcessor.compute_logprobs_for_multi_item_scoring` step for logprobs for multi item scoring
```python
    def compute_logprobs_for_multi_item_scoring(
        self,
        input_ids,
        hidden_states,
        lm_head: VocabParallelEmbedding,
        logits_metadata: Union[LogitsMetadata, ForwardBatch],
        multi_item_delimiter_indices: List[torch.Tensor],
    ):
        """
        Compute logprobs for multi-item scoring using pre-computed delimiter indices.

        Sequence format: Query<delimiter>Item1<delimiter>Item2<delimiter>...
        Scoring positions: Extracts logprobs at positions before each <delimiter>

        Args:
            input_ids: Input token IDs. Shape: [total_sequence_length].
            hidden_states: Hidden states from the model. Shape: [sequence_length, hidden_dim].
            lm_head: Language model head for computing logits.
            logits_metadata: Metadata containing batch info and logprob specs.
            multi_item_delimiter_indices: Pre-computed delimiter positions per request (CPU tensors).
        """
        # Compute positions just before each delimiter.
        # Build offset-adjusted indices on CPU, then do a single CPU→GPU transfer.
        device = input_ids.device
        all_tensors = []
        if logits_metadata.extend_seq_lens_cpu is not None:
            offset = 0
            for req_seq_len, indices_tensor in zip(
                logits_metadata.extend_seq_lens_cpu, multi_item_delimiter_indices
            ):
                if len(indices_tensor) > 0:
                    # Note: if the first delimiter is at position 0 (empty query),
                    # indices - 1 wraps to -1. This is harmless — the first
                    # delimiter entry is always discarded by
                    # _process_multi_item_scoring_results.
                    all_tensors.append(indices_tensor + (offset - 1))
                offset += req_seq_len
        else:
            all_tensors.append(multi_item_delimiter_indices[0] - 1)
        multi_item_indices = torch.cat(all_tensors).to(device, non_blocking=True)

        # Extract hidden states at delimiter positions for multi-item scoring
        sliced_hidden = hidden_states[multi_item_indices]

        sliced_logits = self._get_logits(sliced_hidden, lm_head, logits_metadata)
        sliced_logprobs = torch.nn.functional.log_softmax(sliced_logits, dim=-1)

        # Initialize return values
        input_token_ids_logprobs_val = []
        input_token_ids_logprobs_idx = []
        input_top_logprobs_val = None
        input_top_logprobs_idx = None

        # Recalculate extend_logprob_pruned_lens_cpu to match delimiter counts per request
        if (
            logits_metadata.token_ids_logprobs
            or logits_metadata.extend_return_top_logprob
        ):
            logits_metadata.extend_logprob_pruned_lens_cpu = [
                len(t) for t in multi_item_delimiter_indices
            ]

        # Get the logprobs of specified token ids
        if logits_metadata.extend_token_ids_logprob:
            (
                input_token_ids_logprobs_val,
                input_token_ids_logprobs_idx,
            ) = get_token_ids_logprobs_prefill(
                sliced_logprobs, logits_metadata, no_copy_to_cpu=True
            )

        # Get the logprob of top-k tokens
        if logits_metadata.extend_return_top_logprob:
            (
                input_top_logprobs_val,
                input_top_logprobs_idx,
            ) = get_top_logprobs_prefill(sliced_logprobs, logits_metadata)

        # MIS scores come from input_token_ids_logprobs_val (label-token logprobs),
        # not from per-position input_token_logprobs. However, the shared logprob
        # pipeline (add_input_logprob_return_values) asserts input_token_logprobs is
        # non-None, converts it to a tuple, slices it, and validates its length —
        # all before score_request() ever sees the result. We can't set it to None
        # without changing those shared asserts, so we fill with zeros to satisfy
        # the pipeline. score_request() ignores this field entirely.
        input_token_logprobs = torch.zeros(multi_item_indices.shape[0], device=device)

        return LogitsProcessorOutput(
            next_token_logits=None,
            input_token_logprobs=input_token_logprobs,
            input_top_logprobs_val=input_top_logprobs_val,
            input_top_logprobs_idx=input_top_logprobs_idx,
            input_token_ids_logprobs_val=input_token_ids_logprobs_val,
            input_token_ids_logprobs_idx=input_token_ids_logprobs_idx,
            # FIXME: These fields are not logits-related but are passed through here as a
            # workaround since ForwardBatch is local to forward_batch_generation().
            # They should be moved to GenerationBatchResult to keep this class clean.
            mm_input_embeds=logits_metadata.mm_input_embeds,
        )
```
**EN:** This block defines `LogitsProcessor.compute_logprobs_for_multi_item_scoring` and contains the main logic for this step. It mainly invokes `torch.cat.to`, `self._get_logits`, `torch.nn.functional.log_softmax`, `torch.zeros`, and `LogitsProcessorOutput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `device`, `all_tensors`, `multi_item_indices`, `sliced_hidden`, and `sliced_logits` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LogitsProcessor.compute_logprobs_for_multi_item_scoring`，并承载这一阶段的核心逻辑。 它主要调用 `torch.cat.to`、`self._get_logits`、`torch.nn.functional.log_softmax`、`torch.zeros` 以及 `LogitsProcessorOutput`，说明该流程会编排底层辅助函数或计算内核。 像 `device`、`all_tensors`、`multi_item_indices`、`sliced_hidden` 以及 `sliced_logits` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1113-1139: Function `fused_softcap_kernel` and its core logic
```python
@triton.jit
def fused_softcap_kernel(
    full_logits_ptr,
    softcapping_value,
    ncols,
    row_stride,
    BLOCK_SIZE: tl.constexpr,
):
    row = tl.program_id(1).to(tl.int64)
    pid = tl.program_id(0).to(tl.int64)
    block_start = pid * BLOCK_SIZE
    offsets = block_start + tl.arange(0, BLOCK_SIZE)
    mask = offsets < ncols

    # Load values
    row_ptr = full_logits_ptr + row * row_stride
    x = tl.load(row_ptr + offsets, mask=mask)

    # Perform operations in-place
    x = x / softcapping_value
    x = libdevice.tanh(x)
    x = x * softcapping_value

    # Store result
    tl.store(row_ptr + offsets, x, mask=mask)
```
**EN:** This block defines `fused_softcap_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id.to`, `tl.load`, `libdevice.tanh`, `tl.store`, and `tl.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `row`, `pid`, `block_start`, `offsets`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fused_softcap_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id.to`、`tl.load`、`libdevice.tanh`、`tl.store` 以及 `tl.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `row`、`pid`、`block_start`、`offsets` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 1140-1162: Function `fused_softcap` and its core logic
```python
def fused_softcap(full_logits, final_logit_softcapping):
    if full_logits.is_contiguous():
        nrows, ncols = 1, full_logits.numel()
        row_stride = ncols
    else:
        assert full_logits.ndim == 2, "non-contiguous softcap requires 2D tensor"
        assert (
            full_logits.stride(1) == 1
        ), "non-contiguous softcap requires contiguous columns"
        nrows, ncols = full_logits.shape
        row_stride = full_logits.stride(0)

    BLOCK_SIZE = 1024
    grid = ((ncols + BLOCK_SIZE - 1) // BLOCK_SIZE, nrows)

    fused_softcap_kernel[grid](
        full_logits_ptr=full_logits,
        softcapping_value=final_logit_softcapping,
        ncols=ncols,
        row_stride=row_stride,
        BLOCK_SIZE=BLOCK_SIZE,
    )
    return full_logits
```
**EN:** This block defines `fused_softcap` and contains the main logic for this step. It mainly invokes `full_logits.is_contiguous`, `fused_softcap_kernel`, `full_logits.stride`, and `full_logits.numel`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `BLOCK_SIZE`, `grid`, `nrows`, `ncols`, and `row_stride` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_softcap`，并承载这一阶段的核心逻辑。 它主要调用 `full_logits.is_contiguous`、`fused_softcap_kernel`、`full_logits.stride` 以及 `full_logits.numel`，说明该流程会编排底层辅助函数或计算内核。 像 `BLOCK_SIZE`、`grid`、`nrows`、`ncols` 以及 `row_stride` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `LogitsProcessorOutput`, `LogitsMetadata`, `LogitsProcessor`, `fused_softcap_kernel`, and `fused_softcap`. / **主要符号**：核心入口包括 `LogitsProcessorOutput`、`LogitsMetadata`、`LogitsProcessor`、`fused_softcap_kernel` 以及 `fused_softcap`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `dataclasses`, `logging`, `typing.Any`, `typing.Dict`, `typing.List`, `typing.Optional`, `typing.Tuple`, and `typing.Union` / **标准库**：`dataclasses`、`logging`、`typing.Any`、`typing.Dict`、`typing.List`、`typing.Optional`、`typing.Tuple` 以及 `typing.Union`
- **Third-party**: `torch`, `triton`, `triton.language`, `torch.nn`, and `triton.language.extra.libdevice` / **第三方依赖**：`torch`、`triton`、`triton.language`、`torch.nn` 以及 `triton.language.extra.libdevice`
- **Internal SGLang modules**: `sglang.srt.distributed.get_tensor_model_parallel_world_size`, `sglang.srt.distributed.tensor_model_parallel_all_gather`, `sglang.srt.environ.envs`, `sglang.srt.layers.dp_attention.DpPaddingMode`, `sglang.srt.layers.dp_attention.attn_tp_all_gather`, `sglang.srt.layers.dp_attention.attn_tp_all_gather_into_tensor`, `sglang.srt.layers.dp_attention.dp_gather_replicate`, `sglang.srt.layers.dp_attention.dp_scatter`, `sglang.srt.layers.dp_attention.get_attention_dp_rank`, `sglang.srt.layers.dp_attention.get_attention_dp_size`, `sglang.srt.layers.dp_attention.get_attention_tp_size`, and `sglang.srt.layers.dp_attention.get_dp_device` / **SGLang 内部模块**：`sglang.srt.distributed.get_tensor_model_parallel_world_size`、`sglang.srt.distributed.tensor_model_parallel_all_gather`、`sglang.srt.environ.envs`、`sglang.srt.layers.dp_attention.DpPaddingMode`、`sglang.srt.layers.dp_attention.attn_tp_all_gather`、`sglang.srt.layers.dp_attention.attn_tp_all_gather_into_tensor`、`sglang.srt.layers.dp_attention.dp_gather_replicate`、`sglang.srt.layers.dp_attention.dp_scatter`、`sglang.srt.layers.dp_attention.get_attention_dp_rank`、`sglang.srt.layers.dp_attention.get_attention_dp_size`、`sglang.srt.layers.dp_attention.get_attention_tp_size` 以及 `sglang.srt.layers.dp_attention.get_dp_device`
