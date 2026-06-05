# speculator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/spec_decode/eagle/speculator.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `EagleSpeculator`, `_prepare_eagle_inputs_kernel`, `prepare_eagle_inputs` for the V1 `worker/gpu/spec_decode/eagle` subsystem. / 为 V1 的 `worker/gpu/spec_decode/eagle` 子系统实现 `EagleSpeculator`, `_prepare_eagle_inputs_kernel`, `prepare_eagle_inputs`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from typing import Any

import torch
import torch.nn as nn

from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.config.compilation import CUDAGraphMode
from vllm.forward_context import BatchDescriptor, set_forward_context
from vllm.logger import init_logger
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.triton_utils import tl, triton
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.worker.gpu.attn_utils import (
    build_attn_metadata,
    build_slot_mappings_by_layer,
    init_attn_backend,
)
from vllm.v1.worker.gpu.block_table import BlockTables
from vllm.v1.worker.gpu.cudagraph_utils import (
    BatchExecutionDescriptor,
    CapturedAttentionState,
    get_uniform_token_count,
)
from vllm.v1.worker.gpu.dp_utils import dispatch_cg_and_sync_dp
from vllm.v1.worker.gpu.input_batch import InputBatch, InputBuffers
from vllm.v1.worker.gpu.model_states.interface import ModelState
from vllm.v1.worker.gpu.sample.gumbel import gumbel_sample
from vllm.v1.worker.gpu.spec_decode.eagle.cudagraph import (
    DecodeEagleCudaGraphManager,
    PrefillEagleCudaGraphManager,
)
from vllm.v1.worker.gpu.spec_decode.eagle.utils import load_eagle_model

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `EagleSpeculator` class / `EagleSpeculator` 类
```python
class EagleSpeculator:
```
**EN:** Introduces the `EagleSpeculator` class. Core methods include `__init__`, `init_cudagraph_manager`, `load_model`, `set_attn`, `run_model`, `_sample_draft`.
**CN:** 这里定义 `EagleSpeculator` 类。核心方法包括 `__init__`, `init_cudagraph_manager`, `load_model`, `set_attn`, `run_model`, `_sample_draft`。

### `EagleSpeculator.__init__` method / `EagleSpeculator.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig, device: torch.device):
        self.vllm_config = vllm_config
        self.device = device

        self.speculative_config = vllm_config.speculative_config
        assert self.speculative_config is not None
        self.method = self.speculative_config.method
        self.num_speculative_steps = self.speculative_config.num_speculative_tokens
        self.draft_model_config = self.speculative_config.draft_model_config

        self.scheduler_config = vllm_config.scheduler_config
        self.max_num_reqs = self.scheduler_config.max_num_seqs
        self.max_num_tokens = self.scheduler_config.max_num_batched_tokens
        self.max_model_len = vllm_config.model_config.max_model_len
        # We need to get the hidden size from the draft model config because
        # the draft model's hidden size can be different from the target model's
        # hidden size (e.g., Llama 3.3 70B).
        self.hidden_size = self.draft_model_config.get_hidden_size()
        # Widen for HC-multiplexed residuals (e.g. DeepSeek V4 feeds the MTP
        # draft the target's pre-hc_head (T, hc_mult * hidden_size) residual).
        # Non-HC models default to hc_mult=1 and are unaffected.
        hc_mult = getattr(self.draft_model_config.hf_config, "hc_mult", 1)
        self.hidden_size = self.hidden_size * hc_mult
        self.vocab_size = self.draft_model_config.get_vocab_size()
        self.dtype = vllm_config.model_config.dtype
        self.use_fp64_gumbel = vllm_config.model_config.use_fp64_gumbel

        # DP configuration
        self.dp_size = vllm_config.parallel_config.data_parallel_size
        self.dp_rank = vllm_config.parallel_config.data_parallel_rank

        self.input_buffers = InputBuffers(
            max_num_reqs=self.max_num_reqs,
            max_num_tokens=self.max_num_tokens,
            device=device,
        )
        self.hidden_states = torch.zeros(
            self.max_num_tokens, self.hidden_size, dtype=self.dtype, device=device
        )
        self.idx_mapping = torch.zeros(
            self.max_num_reqs, dtype=torch.int32, device=device
        )
        self.temperature = torch.zeros(
            self.max_num_reqs, dtype=torch.float32, device=device
        )
        self.seeds = torch.zeros(self.max_num_reqs, dtype=torch.int64, device=device)
        self.draft_tokens = torch.zeros(
            self.max_num_reqs,
            self.num_speculative_steps,
            dtype=torch.int64,
            device=device,
        )
        self.current_draft_step = torch.tensor(0, dtype=torch.int64, device=device)
        self.last_token_indices = torch.zeros(
            self.max_num_reqs, dtype=torch.int64, device=device
        )
        self.arange = torch.arange(
            self.max_num_reqs + 1, dtype=torch.int32, device="cpu"
        )

        self.supports_mm_inputs = MULTIMODAL_REGISTRY.supports_multimodal_inputs(
            self.draft_model_config
        )
        if self.supports_mm_inputs:
            self.inputs_embeds = torch.zeros(
                self.max_num_tokens, self.hidden_size, dtype=self.dtype, device=device
            )

        self.draft_logits: torch.Tensor | None = None
        if self.speculative_config.draft_sample_method == "probabilistic":
            self.draft_logits = torch.zeros(
                self.max_num_reqs,
                self.num_speculative_steps,
                self.vocab_size,
                dtype=torch.float32,
                device=device,
            )

        self.prefill_cudagraph_manager: PrefillEagleCudaGraphManager | None = None
        self.decode_cudagraph_manager: DecodeEagleCudaGraphManager | None = None
```
**EN:** This method initializes the object state within `EagleSpeculator`. Key calls include `get_hidden_size`, `getattr`, `get_vocab_size`, `InputBuffers`, `zeros`, `tensor`. It touches state such as `vllm_config`, `device`, `speculative_config`, `method`, `num_speculative_steps`, `draft_model_config`, `scheduler_config`, `max_num_reqs`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`EagleSpeculator`。 关键调用包括 `get_hidden_size`, `getattr`, `get_vocab_size`, `InputBuffers`, `zeros`, `tensor`。 它会读写 `vllm_config`, `device`, `speculative_config`, `method`, `num_speculative_steps`, `draft_model_config`, `scheduler_config`, `max_num_reqs` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EagleSpeculator.run_model` method / `EagleSpeculator.run_model` 方法
```python
    @torch.inference_mode()
    def run_model(
        self,
        num_tokens: int,
        attn_metadata: dict[str, Any] | None,
        slot_mappings: dict[str, torch.Tensor] | None,
        num_tokens_across_dp: torch.Tensor | None,
        cudagraph_runtime_mode: CUDAGraphMode = CUDAGraphMode.NONE,
        mm_inputs: tuple[list[torch.Tensor], torch.Tensor] | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        batch_descriptor = BatchDescriptor(num_tokens=num_tokens)
        with set_forward_context(
            attn_metadata,
            self.vllm_config,
            num_tokens=num_tokens,
            cudagraph_runtime_mode=cudagraph_runtime_mode,
            num_tokens_across_dp=num_tokens_across_dp,
            slot_mapping=slot_mappings,
            batch_descriptor=batch_descriptor,
        ):
            inputs_embeds = None
            if self.supports_mm_inputs:
                # Merge multimodal embeddings with input ids.
                mm_embeds, is_mm_embed = mm_inputs or (None, None)
                num_input_tokens = (
                    is_mm_embed.shape[0] if is_mm_embed is not None else num_tokens
                )
                self.inputs_embeds[:num_input_tokens] = self.model.embed_input_ids(
                    self.input_buffers.input_ids[:num_input_tokens],
                    multimodal_embeddings=mm_embeds,
                    is_multimodal=is_mm_embed,
                )
                inputs_embeds = self.inputs_embeds[:num_tokens]

            ret_hidden_states = self.model(
                input_ids=self.input_buffers.input_ids[:num_tokens],
                positions=self.input_buffers.positions[:num_tokens],
                hidden_states=self.hidden_states[:num_tokens],
                inputs_embeds=inputs_embeds,
            )
        if self.method == "mtp":
            last_hidden_states = ret_hidden_states
            hidden_states = ret_hidden_states
        else:
            last_hidden_states, hidden_states = ret_hidden_states
        return last_hidden_states, hidden_states
```
**EN:** This method runs the main execution path within `EagleSpeculator`. Key calls include `inference_mode`, `BatchDescriptor`, `set_forward_context`, `model`, `embed_input_ids`. It touches state such as `inputs_embeds`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`EagleSpeculator`。 关键调用包括 `inference_mode`, `BatchDescriptor`, `set_forward_context`, `model`, `embed_input_ids`。 它会读写 `inputs_embeds` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EagleSpeculator.multi_step_decode` method / `EagleSpeculator.multi_step_decode` 方法
```python
    def multi_step_decode(
        self,
        num_reqs: int,
        skip_attn: bool,
        batch_desc: BatchExecutionDescriptor,
        num_tokens_across_dp: torch.Tensor | None,
    ) -> None:
        positions = self.input_buffers.positions[:num_reqs]
        query_start_loc = self.input_buffers.query_start_loc[: num_reqs + 1]
        idx_mapping = self.idx_mapping[:num_reqs]

        for step in range(1, self.num_speculative_steps):
            attn_metadata = None
            slot_mappings_by_layer = None
            if not skip_attn:
                # Build attention metadata and slot mappings for each draft
                # decode step. It is necessary to rebuild the attention
                # metadata even when replaying the FULL graph so that any
                # attention metadata builder state is updated.
                slot_mappings = self.block_tables.compute_slot_mappings(
                    idx_mapping,
                    query_start_loc,
                    positions,
                    batch_desc.num_tokens,
                )
                slot_mappings_by_layer = build_slot_mappings_by_layer(
                    slot_mappings, self.kv_cache_config
                )
                attn_metadata = self._build_draft_attn_metadata(
                    num_reqs=num_reqs,
                    num_reqs_padded=batch_desc.num_reqs or num_reqs,
                    num_tokens_padded=batch_desc.num_tokens,
                )

            # Update the current draft step.
            self.current_draft_step.fill_(step)

            # Generate draft tokens for the current step.
            if batch_desc.cg_mode == CUDAGraphMode.FULL:
                assert self.decode_cudagraph_manager is not None
                self.decode_cudagraph_manager.run_fullgraph(batch_desc)
            else:
                self.generate_draft(
                    num_reqs,
                    batch_desc.num_tokens,
                    attn_metadata,
                    slot_mappings_by_layer,
                    num_tokens_across_dp=num_tokens_across_dp,
                    cudagraph_runtime_mode=batch_desc.cg_mode,
                )
```
**EN:** This method implements `multi_step_decode` within `EagleSpeculator`. Key calls include `range`, `fill_`, `compute_slot_mappings`, `build_slot_mappings_by_layer`, `_build_draft_attn_metadata`, `run_fullgraph`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `multi_step_decode`，其作用域位于`EagleSpeculator`。 关键调用包括 `range`, `fill_`, `compute_slot_mappings`, `build_slot_mappings_by_layer`, `_build_draft_attn_metadata`, `run_fullgraph`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_prepare_eagle_inputs_kernel` function / `_prepare_eagle_inputs_kernel` 函数
```python
@triton.jit
def _prepare_eagle_inputs_kernel(
    last_token_indices_ptr,
    eagle_current_draft_step_ptr,
    eagle_input_ids_ptr,
    eagle_positions_ptr,
    eagle_query_start_loc_ptr,
    eagle_seq_lens_ptr,
    target_input_ids_ptr,
    target_positions_ptr,
    idx_mapping_ptr,
    last_sampled_ptr,
    next_prefill_tokens_ptr,
    num_sampled_ptr,
    num_rejected_ptr,
    query_start_loc_ptr,
    seq_lens_ptr,
    max_num_reqs,
    BLOCK_SIZE: tl.constexpr,
):
    req_idx = tl.program_id(0)
    num_reqs = tl.num_programs(0)
    req_state_idx = tl.load(idx_mapping_ptr + req_idx)

    query_start = tl.load(query_start_loc_ptr + req_idx)
    query_end = tl.load(query_start_loc_ptr + req_idx + 1)
    query_len = query_end - query_start
    seq_len = tl.load(seq_lens_ptr + req_idx)

    # Get the true query length and next token after accounting for rejected tokens.
    num_rejected = tl.load(num_rejected_ptr + req_idx)
    query_len -= num_rejected

    num_sampled = tl.load(num_sampled_ptr + req_idx)
    if num_sampled > 0:
        next_token = tl.load(last_sampled_ptr + req_state_idx).to(tl.int32)
    else:
        # Chunked prefilling.
        # Get the next prefill token.
        next_token = tl.load(next_prefill_tokens_ptr + req_state_idx)

    # Shift target_input_ids by one.
    for i in range(1, query_len, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < query_len
        input_ids = tl.load(target_input_ids_ptr + query_start + block, mask=mask)
        tl.store(eagle_input_ids_ptr + query_start + block - 1, input_ids, mask=mask)

    last_token_index = query_start + query_len - 1
    tl.store(last_token_indices_ptr + req_idx, last_token_index)
    tl.store(eagle_input_ids_ptr + last_token_index, next_token)

    # Copy positions.
    for i in range(0, query_len, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < query_len
        target_pos = tl.load(target_positions_ptr + query_start + block, mask=mask)
        tl.store(eagle_positions_ptr + query_start + block, target_pos, mask=mask)

    # Copy query start locations.
    tl.store(eagle_query_start_loc_ptr + req_idx, query_start)
    # Copy sequence lengths.
    tl.store(eagle_seq_lens_ptr + req_idx, seq_len)
    if req_idx == (num_reqs - 1):
        # Reset the current draft step to 0.
        tl.store(eagle_current_draft_step_ptr, 0)
        # Pad query_start_loc for CUDA graphs.
        for i in range(num_reqs, max_num_reqs + 1, BLOCK_SIZE):
            block = i + tl.arange(0, BLOCK_SIZE)
            mask = block < max_num_reqs + 1
            tl.store(eagle_query_start_loc_ptr + block, query_end, mask=mask)
        # Pad seq_lens for CUDA graphs.
        for i in range(num_reqs, max_num_reqs, BLOCK_SIZE):
            block = i + tl.arange(0, BLOCK_SIZE)
            mask = block < max_num_reqs
            tl.store(eagle_seq_lens_ptr + block, 0, mask=mask)
        # Pad last_token_indices for CUDA graphs.
        for i in range(num_reqs, max_num_reqs, BLOCK_SIZE):
            block = i + tl.arange(0, BLOCK_SIZE)
            mask = block < max_num_reqs
            tl.store(last_token_indices_ptr + block, 0, mask=mask)
```
**EN:** This function implements `_prepare_eagle_inputs_kernel` within the module. Key calls include `program_id`, `num_programs`, `load`, `range`, `store`, `to`. The control flow contains 2 branch(es) and 5 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_prepare_eagle_inputs_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `num_programs`, `load`, `range`, `store`, `to`。 控制流包含 2 个分支和 5 个循环，说明这里承担了较强的协调逻辑。

### `prepare_eagle_inputs` function / `prepare_eagle_inputs` 函数
```python
def prepare_eagle_inputs(
    # [num_reqs]
    last_token_indices: torch.Tensor,
    current_draft_step: torch.Tensor,
    input_buffers: InputBuffers,
    input_batch: InputBatch,
    # [num_reqs]
    num_sampled: torch.Tensor,
    # [num_reqs]
    num_rejected: torch.Tensor,
    # [max_num_reqs]
    last_sampled: torch.Tensor,
    # [max_num_reqs]
    next_prefill_tokens: torch.Tensor,
    max_num_reqs,
) -> torch.Tensor:
    num_reqs = input_batch.num_reqs
    _prepare_eagle_inputs_kernel[(num_reqs,)](
        last_token_indices,
        current_draft_step,
        input_buffers.input_ids,
        input_buffers.positions,
        input_buffers.query_start_loc,
        input_buffers.seq_lens,
        input_batch.input_ids,
        input_batch.positions,
        input_batch.idx_mapping,
        last_sampled,
        next_prefill_tokens,
        num_sampled,
        num_rejected,
        input_batch.query_start_loc,
        input_batch.seq_lens,
        max_num_reqs,
        BLOCK_SIZE=1024,
    )
    return last_token_indices
```
**EN:** This function prepares inputs and state within the module.
**CN:** 该函数会准备输入与状态，其作用域位于the module。

### `_prepare_eagle_decode_kernel` function / `_prepare_eagle_decode_kernel` 函数
```python
@triton.jit
def _prepare_eagle_decode_kernel(
    draft_tokens_ptr,
    draft_tokens_stride,
    target_seq_lens_ptr,
    num_rejected_ptr,
    input_ids_ptr,
    positions_ptr,
    query_start_loc_ptr,
    seq_lens_ptr,
    max_model_len,
    max_num_reqs,
    BLOCK_SIZE: tl.constexpr,
):
    req_idx = tl.program_id(0)
    num_reqs = tl.num_programs(0) - 1
    if req_idx == num_reqs:
        # Compute query_start_loc. Pad it with the last query_start_loc
        # for CUDA graphs.
        for i in range(0, max_num_reqs + 1, BLOCK_SIZE):
            block = i + tl.arange(0, BLOCK_SIZE)
            q = tl.where(block < num_reqs, block, num_reqs)
            mask = block < max_num_reqs + 1
            tl.store(query_start_loc_ptr + block, q, mask=mask)
        # Pad seq_lens for CUDA graphs.
        for i in range(req_idx, max_num_reqs, BLOCK_SIZE):
            block = i + tl.arange(0, BLOCK_SIZE)
            mask = block < max_num_reqs
            tl.store(seq_lens_ptr + block, 0, mask=mask)
        return

    # draft token -> input id.
    draft_token = tl.load(draft_tokens_ptr + req_idx * draft_tokens_stride)
    tl.store(input_ids_ptr + req_idx, draft_token)

    # Compute position and seq_lens.
    # NOTE(woosuk): To prevent out-of-range access, we clamp these values
    # if they reach the max model length.
    position = tl.load(positions_ptr + req_idx)
    position = tl.minimum(position + 1, max_model_len - 1)
    tl.store(positions_ptr + req_idx, position)

    target_seq_len = tl.load(target_seq_lens_ptr + req_idx)
    num_rejected = tl.load(num_rejected_ptr + req_idx)
    seq_len = target_seq_len - num_rejected
    seq_len = tl.minimum(seq_len + 1, max_model_len)
    tl.store(seq_lens_ptr + req_idx, seq_len)
```
**EN:** This function implements `_prepare_eagle_decode_kernel` within the module. Key calls include `program_id`, `load`, `store`, `minimum`, `num_programs`, `range`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_prepare_eagle_decode_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `store`, `minimum`, `num_programs`, `range`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `prepare_eagle_decode` function / `prepare_eagle_decode` 函数
```python
def prepare_eagle_decode(
    draft_tokens: torch.Tensor,
    target_seq_lens: torch.Tensor,
    num_rejected: torch.Tensor,
    input_buffers: InputBuffers,
    max_model_len: int,
    max_num_reqs: int,
):
    num_reqs = draft_tokens.shape[0]
    _prepare_eagle_decode_kernel[(num_reqs + 1,)](
        draft_tokens,
        draft_tokens.stride(0),
        target_seq_lens,
        num_rejected,
        input_buffers.input_ids,
        input_buffers.positions,
        input_buffers.query_start_loc,
        input_buffers.seq_lens,
        max_model_len,
        max_num_reqs,
        BLOCK_SIZE=1024,
    )
```
**EN:** This function prepares inputs and state within the module. Key calls include `stride`.
**CN:** 该函数会准备输入与状态，其作用域位于the module。 关键调用包括 `stride`。

### `_update_eagle_draft_inputs_kernel` function / `_update_eagle_draft_inputs_kernel` 函数
```python
@triton.jit
def _update_eagle_draft_inputs_kernel(
    output_draft_tokens_ptr,
    output_draft_tokens_stride,
    next_input_hidden_states_ptr,
    next_input_hidden_states_stride,
    input_ids_ptr,
    positions_ptr,
    seq_lens_ptr,
    draft_tokens_ptr,
    current_draft_step_ptr,
    hidden_states_ptr,
    hidden_states_stride,
    hidden_size,
    max_model_len,
    num_speculative_steps,
    BLOCK_SIZE: tl.constexpr,
):
    req_idx = tl.program_id(0)

    # Write the sampled draft token into self.draft_tokens[req_idx, step].
    draft_token = tl.load(draft_tokens_ptr + req_idx)
    step = tl.load(current_draft_step_ptr)
    tl.store(
        output_draft_tokens_ptr + req_idx * output_draft_tokens_stride + step,
        draft_token,
    )

    if step >= num_speculative_steps - 1:
        # This is the final step. Skip updating draft forward inputs.
        return

    # Write the sampled draft token into the input ids tensor for the next
    # forward pass.
    tl.store(input_ids_ptr + req_idx, draft_token)

    # Copy hidden states into the input hidden states tensor for the next
    # forward pass.
    for i in range(0, hidden_size, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < hidden_size
        hidden_states = tl.load(
            hidden_states_ptr + req_idx * hidden_states_stride + block,
            mask=mask,
        )
        tl.store(
            next_input_hidden_states_ptr
            + req_idx * next_input_hidden_states_stride
            + block,
            hidden_states,
            mask=mask,
        )

    # Increment position and seq_lens.
    # NOTE(woosuk): To prevent out-of-range access, we clamp these values
    # if they reach the max model length.
    position = tl.load(positions_ptr + req_idx)
    position = tl.minimum(position + 1, max_model_len - 1)
    tl.store(positions_ptr + req_idx, position)

    seq_len = tl.load(seq_lens_ptr + req_idx)
    seq_len = tl.minimum(seq_len + 1, max_model_len)
    tl.store(seq_lens_ptr + req_idx, seq_len)
```
**EN:** This function implements `_update_eagle_draft_inputs_kernel` within the module. Key calls include `program_id`, `load`, `store`, `range`, `minimum`, `arange`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_update_eagle_draft_inputs_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `store`, `range`, `minimum`, `arange`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `update_eagle_draft_inputs` function / `update_eagle_draft_inputs` 函数
```python
def update_eagle_draft_inputs(
    draft_tokens: torch.Tensor,
    current_draft_step: torch.Tensor,
    hidden_states: torch.Tensor,
    output_draft_tokens: torch.Tensor,
    next_input_hidden_states: torch.Tensor,
    input_buffers: InputBuffers,
    num_reqs: int,
    max_model_len: int,
    num_speculative_steps: int,
):
    _, hidden_size = hidden_states.shape
    _update_eagle_draft_inputs_kernel[(num_reqs,)](
        output_draft_tokens,
        output_draft_tokens.stride(0),
        next_input_hidden_states,
        next_input_hidden_states.stride(0),
        input_buffers.input_ids,
        input_buffers.positions,
        input_buffers.seq_lens,
        draft_tokens,
        current_draft_step,
        hidden_states,
        hidden_states.stride(0),
        hidden_size,
        max_model_len,
        num_speculative_steps,
        BLOCK_SIZE=1024,
    )
```
**EN:** This function updates existing state within the module. Key calls include `stride`.
**CN:** 该函数会更新现有状态，其作用域位于the module。 关键调用包括 `stride`。

## Key Concepts / 关键概念
- `EagleSpeculator`: central class or interface in this module. / `EagleSpeculator`：本模块中的核心类或接口。
- `_prepare_eagle_inputs_kernel`: top-level helper or orchestration entry point. / `_prepare_eagle_inputs_kernel`：顶层辅助函数或编排入口。
- `prepare_eagle_inputs`: top-level helper or orchestration entry point. / `prepare_eagle_inputs`：顶层辅助函数或编排入口。
- `_prepare_eagle_decode_kernel`: top-level helper or orchestration entry point. / `_prepare_eagle_decode_kernel`：顶层辅助函数或编排入口。
- `prepare_eagle_decode`: top-level helper or orchestration entry point. / `prepare_eagle_decode`：顶层辅助函数或编排入口。
- `_update_eagle_draft_inputs_kernel`: top-level helper or orchestration entry point. / `_update_eagle_draft_inputs_kernel`：顶层辅助函数或编排入口。
- `update_eagle_draft_inputs`: top-level helper or orchestration entry point. / `update_eagle_draft_inputs`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.compilation`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.layers.attention_layer_base`, `vllm.multimodal`, `vllm.triton_utils`, `vllm.v1.kv_cache_interface`, `vllm.v1.worker.gpu.attn_utils`, `vllm.v1.worker.gpu.block_table`, `vllm.v1.worker.gpu.cudagraph_utils`, `vllm.v1.worker.gpu.dp_utils`
