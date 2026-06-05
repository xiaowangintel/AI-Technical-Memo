# ascend_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/attention/ascend_backend.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-37: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, List, Optional

import torch
import torch_npu
from sgl_kernel_npu.attention.sinks_attention import (
    attention_sinks_prefill_triton,
    attention_sinks_triton,
)

from sglang.srt.configs.model_config import AttentionArch
from sglang.srt.dllm.config import DllmConfig
from sglang.srt.hardware_backend.npu.attention.ascend_torch_native_backend import (
    AscendTorchNativeAttnBackend,
)
from sglang.srt.hardware_backend.npu.attention.mla_preprocess import (
    is_fia_nz,
    is_mla_preprocess_enabled,
)
from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.attention.nsa.utils import is_nsa_enable_prefill_cp
from sglang.srt.layers.dp_attention import get_attention_tp_size
from sglang.srt.layers.radix_attention import AttentionType
from sglang.srt.layers.utils.cp_utils import cp_all_gather_rerange_kv_cache
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.speculative.spec_info import SpecInput
# ... omitted for brevity ...
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner

import logging

import numpy as np
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `dataclasses`, `typing`, `torch`, `torch_npu`, `sgl_kernel_npu.attention.sinks_attention`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `dataclasses`, `typing`, `torch`, `torch_npu`, `sgl_kernel_npu.attention.sinks_attention`。

### Lines 40-44: Function `_reshape_kv_for_fia_nz` / 函数 `_reshape_kv_for_fia_nz`
```python
def _reshape_kv_for_fia_nz(
    tensor: torch.Tensor, num_heads: int, head_dim: int, page_size: int
) -> torch.Tensor:
    """Reshapes a tensor for FIA NZ format."""
    return tensor.view(-1, 1, num_heads * head_dim // 16, page_size, 16)
```
**EN:** This function implements `_reshape_kv_for_fia_nz`. It primarily calls `tensor.view` to complete its work.
**CN:** 该函数实现了 `_reshape_kv_for_fia_nz`。 它主要通过调用 `tensor.view` 来完成任务。

### Lines 47-47: Constants and shared state / 常量与共享状态
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `logger`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `logger`。

### Lines 50-70: Class `ForwardMetadata` declaration / 类 `ForwardMetadata` 声明
```python
@dataclass
class ForwardMetadata:

    # calculated map for kv positions [bs * maxseqlen]
    block_tables: Optional[torch.Tensor] = None

    # mapped block_tables for swa
    block_tables_swa: Optional[torch.Tensor] = None

    # seq len inputs
    extend_seq_lens_cpu_int: Optional[torch.Tensor] = None
    seq_lens_cpu_int: Optional[torch.Tensor] = None
    seq_lens_cpu_list: Optional[List[int]] = None
    seq_lens_list_cumsum: Optional[List[int]] = None
    seq_lens: Optional[torch.Tensor] = None
    actual_seq_lengths_q: Optional[torch.Tensor] = None
    actual_seq_lengths_kv: Optional[torch.Tensor] = None

    # prefix cache
    prefix_lens: Optional[torch.Tensor] = None
    flatten_prefix_block_tables: Optional[torch.Tensor] = None
```
**EN:** This class establishes `ForwardMetadata` as a compact data container for the surrounding logic. The main stored fields include `block_tables`, `block_tables_swa`, `extend_seq_lens_cpu_int`, `seq_lens_cpu_int`, `seq_lens_cpu_list`, `seq_lens_list_cumsum`.
**CN:** 该类将 `ForwardMetadata` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `block_tables`, `block_tables_swa`, `extend_seq_lens_cpu_int`, `seq_lens_cpu_int`, `seq_lens_cpu_list`, `seq_lens_list_cumsum`。

### Lines 73-73: Class `AscendAttnMaskBuilder` declaration / 类 `AscendAttnMaskBuilder` 声明
```python
class AscendAttnMaskBuilder:
```
**EN:** This class establishes `AscendAttnMaskBuilder` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `generate_mask_flag`, `generate_attn_mask`, `get_attention_mask_id`, `update_attn_cache`, `get_splitfuse_attn_mask`.
**CN:** 该类将 `AscendAttnMaskBuilder` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `generate_mask_flag`, `generate_attn_mask`, `get_attention_mask_id`, `update_attn_cache`, `get_splitfuse_attn_mask` 等方法。

### Lines 74-109: Method `AscendAttnMaskBuilder.__init__` / 方法 `AscendAttnMaskBuilder.__init__`
```python
    def __init__(self, model_runner: ModelRunner, device, use_fia, use_mla):
        """
        Initialize the AscendAttnMaskBuilder class.

        :param model_runner: ModelRunner instance for model execution.
        :param device: Device to run the model on (e.g., 'cuda', 'npu').
        :param use_fia: Boolean flag to indicate if environment variable ASCEND_USE_FIA is set to 1.
        """
        self.use_fia = use_fia
        self.model_runner = model_runner
        self.device = device

        # Initialize mask
        mask_len = 128
        self.mask = self.generate_attn_mask(mask_len, "norm", model_runner.dtype).to(
            self.device
        )

        # Initialize FIA mask
        fia_mask_len = 2048
        self.fia_mask = self.generate_mask_flag(fia_mask_len).to(self.device)

        # Initialize MTP mask
        mtp_mask_len = 2048
        self.mtp_mask = self.generate_mask_flag(mtp_mask_len).to(self.device)

        # Initialize mixed chunk mask cache
        mixed_mask_len = 2048
        self.mixed_chunk_attn_mask = self.get_splitfuse_attn_mask(mixed_mask_len)

        if use_mla:
            # Initialize RingMla mask
            ringmla_mask_len = 512
            self.ringmla_mask = self.generate_attn_mask(
                ringmla_mask_len, "norm", torch.bfloat16
            ).to(self.device)
```
**EN:** This method implements `__init__` on `AscendAttnMaskBuilder`. It primarily calls `self.generate_attn_mask.to`, `self.generate_mask_flag.to`, `self.get_splitfuse_attn_mask`, `self.generate_attn_mask`, `self.generate_mask_flag` to complete its work. State updates are written into `self.use_fia`, `self.model_runner`, `self.device`, `mask_len`, `self.mask`, `fia_mask_len`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnMaskBuilder`）实现了 `__init__`。 它主要通过调用 `self.generate_attn_mask.to`, `self.generate_mask_flag.to`, `self.get_splitfuse_attn_mask`, `self.generate_attn_mask`, `self.generate_mask_flag` 来完成任务。 状态更新主要写入 `self.use_fia`, `self.model_runner`, `self.device`, `mask_len`, `self.mask`, `fia_mask_len`。 实现中使用了条件分支。

### Lines 111-123: Method `AscendAttnMaskBuilder.generate_mask_flag` / 方法 `AscendAttnMaskBuilder.generate_mask_flag`
```python
    @staticmethod
    def generate_mask_flag(max_seq_len):
        """
        Generate a mask flag for attention masks.

        :param max_seq_len: Maximum sequence length for the mask.
        :return: A boolean tensor representing the mask flag.
        """
        # Construct lower triangle matrix.
        mask_flag = torch.ones((max_seq_len, max_seq_len), dtype=torch.bool).tril_()
        # Create upper triangle matrix used to mark mask positions.
        mask_flag = ~mask_flag
        return mask_flag
```
**EN:** This method implements `generate_mask_flag` on `AscendAttnMaskBuilder`. It primarily calls `torch.ones.tril_`, `torch.ones` to complete its work. State updates are written into `mask_flag`.
**CN:** 该方法（属于 `AscendAttnMaskBuilder`）实现了 `generate_mask_flag`。 它主要通过调用 `torch.ones.tril_`, `torch.ones` 来完成任务。 状态更新主要写入 `mask_flag`。

### Lines 125-147: Method `AscendAttnMaskBuilder.generate_attn_mask` / 方法 `AscendAttnMaskBuilder.generate_attn_mask`
```python
    @staticmethod
    def generate_attn_mask(max_seq_len, mode, dtype=torch.float16):
        """
        Generate an attention mask.

        :param max_seq_len: Maximum sequence length for the mask.
        :param mode: Mode of the mask ('mix' or 'norm').
        :param dtype: Data type of the mask tensor.
        :return: A tensor representing the attention mask.
        """
        mask_flag = AscendAttnMaskBuilder.generate_mask_flag(max_seq_len)
        if mode == "mix":
            mask_value = (
                float("-inf") if dtype in [torch.float16, torch.bfloat16] else 1
            )
        else:
            mask_value = torch.finfo(torch.float32).min if dtype == torch.float16 else 1
        attn_mask = (
            torch.zeros(size=(max_seq_len, max_seq_len))
            .masked_fill_(mask_flag, mask_value)
            .to(dtype)
        )
        return attn_mask
```
**EN:** This method implements `generate_attn_mask` on `AscendAttnMaskBuilder`. It primarily calls `AscendAttnMaskBuilder.generate_mask_flag`, `torch.zeros.masked_fill_.to`, `float`, `torch.zeros.masked_fill_`, `torch.finfo`, `torch.zeros` to complete its work. State updates are written into `mask_flag`, `attn_mask`, `mask_value`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnMaskBuilder`）实现了 `generate_attn_mask`。 它主要通过调用 `AscendAttnMaskBuilder.generate_mask_flag`, `torch.zeros.masked_fill_.to`, `float`, `torch.zeros.masked_fill_`, `torch.finfo`, `torch.zeros` 来完成任务。 状态更新主要写入 `mask_flag`, `attn_mask`, `mask_value`。 实现中使用了条件分支。

### Lines 149-166: Method `AscendAttnMaskBuilder.get_attention_mask_id` / 方法 `AscendAttnMaskBuilder.get_attention_mask_id`
```python
    @staticmethod
    def get_attention_mask_id(seq_lens, extend_lens):
        """
        Generate attention mask IDs based on sequence lengths and extended lengths.

        :param seq_lens: Sequence lengths.
        :param extend_lens: Extended lengths.
        :return: A tensor containing the attention mask IDs.
        """
        starts = seq_lens - extend_lens
        ends = seq_lens

        # Use torch.stack to stack the start and end indices together
        ranges = torch.stack((starts, ends), dim=-1)

        # Use list comprehension to generate tensors for each range and concatenate them
        attn_mask_id = torch.cat([torch.arange(start, end) for start, end in ranges])
        return attn_mask_id
```
**EN:** This method implements `get_attention_mask_id` on `AscendAttnMaskBuilder`. It primarily calls `torch.stack`, `torch.cat`, `torch.arange` to complete its work. State updates are written into `starts`, `ends`, `ranges`, `attn_mask_id`.
**CN:** 该方法（属于 `AscendAttnMaskBuilder`）实现了 `get_attention_mask_id`。 它主要通过调用 `torch.stack`, `torch.cat`, `torch.arange` 来完成任务。 状态更新主要写入 `starts`, `ends`, `ranges`, `attn_mask_id`。

### Lines 168-191: Method `AscendAttnMaskBuilder.update_attn_cache` / 方法 `AscendAttnMaskBuilder.update_attn_cache`
```python
    def update_attn_cache(
        self,
        seqlen: int,
        mask_cache: torch.Tensor,
        seq_len_cached: int,
        dtype: torch.dtype,
        mode,
    ):
        """
        Update the attention mask cache.

        :param seqlen: Maximum sequence length.
        :param mask_cache: Current attention mask cache.
        :param seq_len_cached: Cached sequence length.
        :param dtype: Data type of the mask tensor.
        :param mode: Mode of the mask ('mix' or 'norm').
        :return: Updated mask cache and sequence length cache.
        """
        if seqlen > seq_len_cached:
            seq_len_cached = seqlen
            mask_cache = self.generate_attn_mask(seqlen, mode, dtype)
        if mask_cache.dtype != dtype:
            mask_cache = mask_cache.to(dtype)
        return mask_cache, seq_len_cached
```
**EN:** This method implements `update_attn_cache` on `AscendAttnMaskBuilder`. It primarily calls `self.generate_attn_mask`, `mask_cache.to` to complete its work. State updates are written into `seq_len_cached`, `mask_cache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnMaskBuilder`）实现了 `update_attn_cache`。 它主要通过调用 `self.generate_attn_mask`, `mask_cache.to` 来完成任务。 状态更新主要写入 `seq_len_cached`, `mask_cache`。 实现中使用了条件分支。

### Lines 193-208: Method `AscendAttnMaskBuilder.get_splitfuse_attn_mask` / 方法 `AscendAttnMaskBuilder.get_splitfuse_attn_mask`
```python
    def get_splitfuse_attn_mask(
        self,
        seq_lens: torch.Tensor = None,
    ) -> torch.Tensor:
        """
        Generate a splitfuse attention mask.

        :param seq_lens: Sequence lengths.
        :return: A tensor representing the splitfuse attention mask.
        """
        attn_mask = (
            torch.triu(torch.ones(seq_lens, seq_lens), diagonal=1)
            .to(torch.int8)
            .to(self.device)
        )
        return attn_mask
```
**EN:** This method implements `get_splitfuse_attn_mask` on `AscendAttnMaskBuilder`. It primarily calls `torch.triu.to.to`, `torch.triu.to`, `torch.triu`, `torch.ones` to complete its work. State updates are written into `attn_mask`.
**CN:** 该方法（属于 `AscendAttnMaskBuilder`）实现了 `get_splitfuse_attn_mask`。 它主要通过调用 `torch.triu.to.to`, `torch.triu.to`, `torch.triu`, `torch.ones` 来完成任务。 状态更新主要写入 `attn_mask`。

### Lines 211-251: Function `_cp_allgather_and_save_kv_npu` / 函数 `_cp_allgather_and_save_kv_npu`
```python
def _cp_allgather_and_save_kv_npu(forward_batch, layer, k, v, cp_size):
    """NPU-compatible CP KV all-gather with merged K/V communication.

    Merges K and V along the feature dimension so only one all-gather is
    needed instead of two, halving communication latency.

    k shape: [S_local, tp_k_head_num, qk_head_dim]
    v shape: [S_local, tp_v_head_num, v_head_dim]

    Equivalent to cp_allgather_and_save_kv_cache() in cp_utils.py, but uses
    a single all-gather for both K and V.
    """
    cache_loc = (
        forward_batch.out_cache_loc
        if not layer.is_cross_attention
        else forward_batch.encoder_out_cache_loc
    )
    # Save original trailing shapes for reshape after gather.
    k_tail = k.shape[1:]  # (tp_k_head_num, qk_head_dim)
    v_tail = v.shape[1:]  # (tp_v_head_num, v_head_dim)

    # Flatten trailing dims then concat → one all-gather instead of two.
    # Works for GQA where tp_k_head_num != tp_v_head_num.
    k_flat = k.contiguous().reshape(k.shape[0], -1)  # [S_local, k_feat]
    v_flat = v.contiguous().reshape(v.shape[0], -1)  # [S_local, v_feat]
    k_feat_size = k_flat.shape[-1]
    kv_flat = torch.cat([k_flat, v_flat], dim=-1)  # [S_local, k_feat + v_feat]

# ... omitted for brevity ...
    forward_batch.token_to_kv_pool.set_kv_buffer(
        layer,
        cache_loc,
        key_cache_full,
        value_cache_full,
    )
```
**EN:** This function implements `_cp_allgather_and_save_kv_npu`. It primarily calls `k.contiguous.reshape`, `v.contiguous.reshape`, `torch.cat`, `cp_all_gather_rerange_kv_cache`, `kv_full.reshape`, `forward_batch.token_to_kv_pool.set_kv_buffer` to complete its work. State updates are written into `cache_loc`, `k_tail`, `v_tail`, `k_flat`, `v_flat`, `k_feat_size`.
**CN:** 该函数实现了 `_cp_allgather_and_save_kv_npu`。 它主要通过调用 `k.contiguous.reshape`, `v.contiguous.reshape`, `torch.cat`, `cp_all_gather_rerange_kv_cache`, `kv_full.reshape`, `forward_batch.token_to_kv_pool.set_kv_buffer` 来完成任务。 状态更新主要写入 `cache_loc`, `k_tail`, `v_tail`, `k_flat`, `v_flat`, `k_feat_size`。

### Lines 254-254: Class `AscendAttnBackend` declaration / 类 `AscendAttnBackend` 声明
```python
class AscendAttnBackend(AttentionBackend):
```
**EN:** This class establishes `AscendAttnBackend` as the main container/coordinator for the surrounding logic. It inherits from `AttentionBackend`. Its core interface includes methods such as `__init__`, `get_verify_buffers_to_fill_after_draft`, `update_verify_buffers_to_fill_after_draft`, `init_forward_metadata`, `init_cuda_graph_state`, `init_forward_metadata_capture_cuda_graph`.
**CN:** 该类将 `AscendAttnBackend` 定义为周边逻辑的主要封装体或协调者。 它继承自 `AttentionBackend`。 其核心接口包括 `__init__`, `get_verify_buffers_to_fill_after_draft`, `update_verify_buffers_to_fill_after_draft`, `init_forward_metadata`, `init_cuda_graph_state`, `init_forward_metadata_capture_cuda_graph` 等方法。

### Lines 256-333: Method `AscendAttnBackend.__init__` / 方法 `AscendAttnBackend.__init__`
```python
    def __init__(self, model_runner: ModelRunner, speculative_step_id: int = 0):
        super().__init__()
        self.forward_metadata = None
        self.device = model_runner.device
        self.speculative_step_id = speculative_step_id
        self.speculative_step_offset_npu = torch.tensor(
            speculative_step_id + 1, device="npu"
        )
        self.page_size = model_runner.page_size
        self.model_dtype = model_runner.model_config.dtype
        self.use_mla = model_runner.model_config.attention_arch == AttentionArch.MLA
        if self.use_mla:
            self.kv_lora_rank = model_runner.model_config.kv_lora_rank
            self.qk_rope_head_dim = model_runner.model_config.qk_rope_head_dim
            if (
                "MiniCPM3ForCausalLM"
                in model_runner.model_config.hf_config.architectures
            ):
                self.qk_nope_head_dim = (
                    model_runner.model_config.hf_config.qk_nope_head_dim
                )
            else:
                self.qk_nope_head_dim = model_runner.model_config.qk_nope_head_dim
            self.q_head_dim = self.qk_rope_head_dim + self.qk_nope_head_dim
        else:
            self.use_alibi = getattr(model_runner.model_config, "use_alibi", False)
            if (
                "Gemma2ForSequenceClassification"
# ... omitted for brevity ...
        self.is_dllm_model = False
        if self.dllm_config is not None:
            self.is_dllm_model = True
            self.dllm_block_size = self.dllm_config.block_size

        self.attn_cp_size = model_runner.attn_cp_size
```
**EN:** This method implements `__init__` on `AscendAttnBackend`. It primarily calls `super.__init__`, `torch.tensor`, `AscendTorchNativeAttnBackend`, `get_bool_env_var`, `AscendAttnMaskBuilder`, `DllmConfig.from_server_args` to complete its work. State updates are written into `self.forward_metadata`, `self.device`, `self.speculative_step_id`, `self.speculative_step_offset_npu`, `self.page_size`, `self.model_dtype`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `torch.tensor`, `AscendTorchNativeAttnBackend`, `get_bool_env_var`, `AscendAttnMaskBuilder`, `DllmConfig.from_server_args` 来完成任务。 状态更新主要写入 `self.forward_metadata`, `self.device`, `self.speculative_step_id`, `self.speculative_step_offset_npu`, `self.page_size`, `self.model_dtype`。 实现中使用了条件分支、迭代逻辑。

### Lines 335-341: Method `AscendAttnBackend.get_verify_buffers_to_fill_after_draft` / 方法 `AscendAttnBackend.get_verify_buffers_to_fill_after_draft`
```python
    def get_verify_buffers_to_fill_after_draft(self):
        """
        Return buffers for verify attention kernels that needs to be filled after draft.

        Typically, these are tree mask and position buffers.
        """
        return [None, None]
```
**EN:** This method implements `get_verify_buffers_to_fill_after_draft` on `AscendAttnBackend`.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `get_verify_buffers_to_fill_after_draft`。

### Lines 343-346: Method `AscendAttnBackend.update_verify_buffers_to_fill_after_draft` / 方法 `AscendAttnBackend.update_verify_buffers_to_fill_after_draft`
```python
    def update_verify_buffers_to_fill_after_draft(
        self, spec_info: SpecInput, cuda_graph_bs: Optional[int]
    ):
        pass
```
**EN:** This method implements `update_verify_buffers_to_fill_after_draft` on `AscendAttnBackend`.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `update_verify_buffers_to_fill_after_draft`。

### Lines 348-435: Method `AscendAttnBackend.init_forward_metadata` / 方法 `AscendAttnBackend.init_forward_metadata`
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Init the metadata for a forward pass."""
        self.forward_metadata = ForwardMetadata()
        seq_lens_max = forward_batch.seq_lens.max()
        if forward_batch.forward_mode.is_target_verify():
            seq_lens_max += self.speculative_num_draft_tokens
        elif (
            forward_batch.forward_mode.is_decode_or_idle()
            and forward_batch.spec_info is not None
        ):
            seq_lens_max += self.speculative_step_id + 1
        self.forward_metadata.block_tables = (
            forward_batch.req_to_token_pool.req_to_token[
                forward_batch.req_pool_indices, :seq_lens_max
            ][:, :: self.page_size]
            // self.page_size
        )
        if self.is_hybrid_swa:
            self.forward_metadata.block_tables_swa = (
                (
                    self.full_to_swa_index_mapping[
                        forward_batch.req_to_token_pool.req_to_token[
                            forward_batch.req_pool_indices, :seq_lens_max
                        ]
                    ][:, :: self.page_size]
                    // self.page_size
                )
                .to(torch.int32)
# ... omitted for brevity ...
                        self.forward_metadata.flatten_prefix_block_tables,
                        torch.flatten(req_prefix_block_tables),
                    )
                )

        self.graph_mode = False
```
**EN:** This method implements `init_forward_metadata` on `AscendAttnBackend`. It primarily calls `ForwardMetadata`, `forward_batch.seq_lens.max`, `forward_batch.forward_mode.is_target_verify`, `forward_batch.seq_lens_cpu.int`, `to.contiguous`, `forward_batch.extend_seq_lens.cpu.int` to complete its work. State updates are written into `self.forward_metadata`, `seq_lens_max`, `self.forward_metadata.block_tables`, `self.forward_metadata.seq_lens_cpu_int`, `self.graph_mode`, `self.forward_metadata.block_tables_swa`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `init_forward_metadata`。 它主要通过调用 `ForwardMetadata`, `forward_batch.seq_lens.max`, `forward_batch.forward_mode.is_target_verify`, `forward_batch.seq_lens_cpu.int`, `to.contiguous`, `forward_batch.extend_seq_lens.cpu.int` 来完成任务。 状态更新主要写入 `self.forward_metadata`, `seq_lens_max`, `self.forward_metadata.block_tables`, `self.forward_metadata.seq_lens_cpu_int`, `self.graph_mode`, `self.forward_metadata.block_tables_swa`。 实现中使用了条件分支、迭代逻辑。

### Lines 437-453: Method `AscendAttnBackend.init_cuda_graph_state` / 方法 `AscendAttnBackend.init_cuda_graph_state`
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        total_context_len = self.max_context_len + self.page_size - 1
        if self.speculative_num_draft_tokens is not None:
            total_context_len += self.speculative_num_draft_tokens
        self.graph_metadata = {
            "block_tables": torch.empty(
                (max_bs, total_context_len // self.page_size),
                dtype=torch.int32,
                device=self.device,
            ),
        }
        if self.is_hybrid_swa:
            self.graph_metadata["block_tables_swa"] = torch.empty(
                (max_bs, total_context_len // self.page_size),
                dtype=torch.int32,
                device=self.device,
            )
```
**EN:** This method implements `init_cuda_graph_state` on `AscendAttnBackend`. It primarily calls `torch.empty` to complete its work. State updates are written into `total_context_len`, `self.graph_metadata`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `init_cuda_graph_state`。 它主要通过调用 `torch.empty` 来完成任务。 状态更新主要写入 `total_context_len`, `self.graph_metadata`。 实现中使用了条件分支。

### Lines 455-549: Method `AscendAttnBackend.init_forward_metadata_capture_cuda_graph` / 方法 `AscendAttnBackend.init_forward_metadata_capture_cuda_graph`
```python
    def init_forward_metadata_capture_cuda_graph(
        self,
        bs: int,
        num_tokens: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[SpecInput],
    ):
        metadata = ForwardMetadata()

        metadata.block_tables = self.graph_metadata["block_tables"][:bs, :]
        if self.is_dllm_model:
            max_len = int(seq_lens[:bs].max().item())
            max_seq_pages = (max_len + self.page_size - 1) // self.page_size
            metadata.block_tables[:bs, :max_seq_pages].copy_(
                (
                    self.req_to_token[req_pool_indices[:bs], :max_len][
                        :, :: self.page_size
                    ]
                    // self.page_size
                ).to(torch.int32)
            )
            metadata.block_tables[:bs, max_seq_pages:].fill_(0)
            metadata.block_tables[bs:, :].fill_(0)

        if self.is_hybrid_swa:
# ... omitted for brevity ...
            )

        self.graph_metadata[bs] = metadata
        self.forward_metadata = metadata

        self.graph_mode = True
```
**EN:** This method implements `init_forward_metadata_capture_cuda_graph` on `AscendAttnBackend`. It primarily calls `ForwardMetadata`, `seq_lens.cpu.int.tolist`, `forward_mode.is_dllm_extend`, `int`, `metadata.block_tables.copy_`, `metadata.block_tables.fill_` to complete its work. State updates are written into `metadata`, `metadata.block_tables`, `metadata.seq_lens_cpu_list`, `metadata.seq_lens`, `self.graph_metadata`, `self.forward_metadata`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `init_forward_metadata_capture_cuda_graph`。 它主要通过调用 `ForwardMetadata`, `seq_lens.cpu.int.tolist`, `forward_mode.is_dllm_extend`, `int`, `metadata.block_tables.copy_`, `metadata.block_tables.fill_` 来完成任务。 状态更新主要写入 `metadata`, `metadata.block_tables`, `metadata.seq_lens_cpu_list`, `metadata.seq_lens`, `self.graph_metadata`, `self.forward_metadata`。 实现中使用了条件分支。

### Lines 551-595: Method `AscendAttnBackend.init_forward_metadata_replay_cuda_graph` / 方法 `AscendAttnBackend.init_forward_metadata_replay_cuda_graph`
```python
    def init_forward_metadata_replay_cuda_graph(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_sum: int,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[SpecInput],
        seq_lens_cpu: Optional[torch.Tensor],
    ):
        metadata = self.graph_metadata[bs]
        max_len = seq_lens_cpu[:bs].max().item()
        if forward_mode.is_target_verify():
            max_len += self.speculative_num_draft_tokens
        elif forward_mode.is_decode_or_idle() and spec_info is not None:
            max_len += self.speculative_step_id + 1
        max_seq_pages = (max_len + self.page_size - 1) // self.page_size

        if self.is_hybrid_swa:
            metadata.block_tables_swa[:bs, :max_seq_pages].copy_(
                self.full_to_swa_index_mapping[
                    self.req_to_token[req_pool_indices[:bs], :max_len]
                ][:, :: self.page_size]
                // self.page_size
            )
            metadata.block_tables_swa[:bs, max_seq_pages:].fill_(0)
            metadata.block_tables_swa[bs:, :].fill_(0)
# ... omitted for brevity ...
            seq_lens = seq_lens + self.speculative_step_offset_npu
        metadata.seq_lens[:bs].copy_(seq_lens[:bs])

        self.forward_metadata = metadata

        self.graph_mode = True
```
**EN:** This method implements `init_forward_metadata_replay_cuda_graph` on `AscendAttnBackend`. It primarily calls `seq_lens_cpu.max.item`, `forward_mode.is_target_verify`, `metadata.block_tables.copy_`, `metadata.block_tables.fill_`, `metadata.seq_lens.copy_`, `metadata.block_tables_swa.copy_` to complete its work. State updates are written into `metadata`, `max_len`, `max_seq_pages`, `self.forward_metadata`, `self.graph_mode`, `seq_lens`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `init_forward_metadata_replay_cuda_graph`。 它主要通过调用 `seq_lens_cpu.max.item`, `forward_mode.is_target_verify`, `metadata.block_tables.copy_`, `metadata.block_tables.fill_`, `metadata.seq_lens.copy_`, `metadata.block_tables_swa.copy_` 来完成任务。 状态更新主要写入 `metadata`, `max_len`, `max_seq_pages`, `self.forward_metadata`, `self.graph_mode`, `seq_lens`。 实现中使用了条件分支。

### Lines 597-598: Method `AscendAttnBackend.get_cuda_graph_seq_len_fill_value` / 方法 `AscendAttnBackend.get_cuda_graph_seq_len_fill_value`
```python
    def get_cuda_graph_seq_len_fill_value(self):
        return 0
```
**EN:** This method implements `get_cuda_graph_seq_len_fill_value` on `AscendAttnBackend`.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `get_cuda_graph_seq_len_fill_value`。

### Lines 600-613: Method `AscendAttnBackend._generate_alibi_bias` / 方法 `AscendAttnBackend._generate_alibi_bias`
```python
    def _generate_alibi_bias(
        self,
        seq_len: int,
        slopes: torch.Tensor,
        num_heads: int,
        device: torch.device,
        dtype: torch.dtype = torch.bfloat16,
    ) -> torch.Tensor:
        position_point = (
            torch.arange(seq_len).view(1, 1, -1).expand(num_heads, -1, -1).to(device)
        )
        alibi = slopes.view(-1, 1, 1) * position_point
        alibi_bias = alibi.view(num_heads, 1, seq_len).to(device).to(dtype)
        return alibi_bias
```
**EN:** This method implements `_generate_alibi_bias` on `AscendAttnBackend`. It primarily calls `torch.arange.view.expand.to`, `alibi.view.to.to`, `slopes.view`, `torch.arange.view.expand`, `alibi.view.to`, `torch.arange.view` to complete its work. State updates are written into `position_point`, `alibi`, `alibi_bias`.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `_generate_alibi_bias`。 它主要通过调用 `torch.arange.view.expand.to`, `alibi.view.to.to`, `slopes.view`, `torch.arange.view.expand`, `alibi.view.to`, `torch.arange.view` 来完成任务。 状态更新主要写入 `position_point`, `alibi`, `alibi_bias`。

### Lines 615-643: Method `AscendAttnBackend.generate_alibi_bias` / 方法 `AscendAttnBackend.generate_alibi_bias`
```python
    def generate_alibi_bias(
        self,
        q_seq_len: int,
        kv_seq_len: int,
        slopes: torch.Tensor,
        num_heads: int,
        device: torch.device,
        is_extend: bool = True,
        dtype: torch.dtype = torch.bfloat16,
    ) -> torch.Tensor:
        MAX_LEN_ALB = 5000
        max_seq_len = max(kv_seq_len, q_seq_len, MAX_LEN_ALB)
        if getattr(self, "alibi_bias", None) is None:
            self.alibi_bias = self._generate_alibi_bias(
                max_seq_len, slopes, num_heads, device, dtype
            )

        if getattr(self, "super_mask", None) is None:
            super_mask = torch.ones(size=(1, max_seq_len, max_seq_len), dtype=dtype)
            super_mask = super_mask.float().fill_(float("-inf")).type_as(super_mask)
            super_mask = torch.triu(super_mask, 1).to(device)
            self.super_mask = super_mask
        if is_extend:
            return (
                self.alibi_bias[:, :q_seq_len, :kv_seq_len]
                + self.super_mask[:, :q_seq_len, :kv_seq_len]
            )
        else:
            return self.alibi_bias[:, :q_seq_len, :kv_seq_len]
```
**EN:** This method implements `generate_alibi_bias` on `AscendAttnBackend`. It primarily calls `max`, `getattr`, `self._generate_alibi_bias`, `torch.ones`, `super_mask.float.fill_.type_as`, `torch.triu.to` to complete its work. State updates are written into `MAX_LEN_ALB`, `max_seq_len`, `self.alibi_bias`, `super_mask`, `self.super_mask`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `generate_alibi_bias`。 它主要通过调用 `max`, `getattr`, `self._generate_alibi_bias`, `torch.ones`, `super_mask.float.fill_.type_as`, `torch.triu.to` 来完成任务。 状态更新主要写入 `MAX_LEN_ALB`, `max_seq_len`, `self.alibi_bias`, `super_mask`, `self.super_mask`。 实现中使用了条件分支。

### Lines 645-713: Method `AscendAttnBackend.attn_alibi` / 方法 `AscendAttnBackend.attn_alibi`
```python
    def attn_alibi(
        self,
        q,
        k_cache,
        v_cache,
        block_tables,
        seq_lens,
        query_lens,
        scale_value,
        num_heads,
        slopes,
        is_extend,
    ):
        curr = 0
        num_prompts = query_lens.shape[0]
        head_size = k_cache.shape[3]
        head_size_v = v_cache.shape[3]
        block_size = k_cache.shape[1]
        attn_output = []
        for i in range(num_prompts):
            seq_len = seq_lens[i].item()
            block_table = block_tables[i]

            j = torch.arange(seq_len, device=block_table.device)

            block_number = block_table[j // block_size]
            block_offset = j % block_size

# ... omitted for brevity ...
            out = out.reshape(-1, num_heads * head_size_v)
            attn_output.append(out)
            curr += q_len
        attn_output = torch.cat(attn_output, dim=0).to(q.dtype).to(q.device)
        attn_output = attn_output.view(-1, num_heads * head_size)
        return attn_output
```
**EN:** This method implements `attn_alibi` on `AscendAttnBackend`. It primarily calls `range`, `torch.cat.to.to`, `attn_output.view`, `seq_lens.item`, `torch.arange`, `k.view` to complete its work. State updates are written into `curr`, `num_prompts`, `head_size`, `head_size_v`, `block_size`, `attn_output`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `attn_alibi`。 它主要通过调用 `range`, `torch.cat.to.to`, `attn_output.view`, `seq_lens.item`, `torch.arange`, `k.view` 来完成任务。 状态更新主要写入 `curr`, `num_prompts`, `head_size`, `head_size_v`, `block_size`, `attn_output`。 实现中使用了条件分支、迭代逻辑。

### Lines 715-783: Method `AscendAttnBackend.do_cp_balance_attn` / 方法 `AscendAttnBackend.do_cp_balance_attn`
```python
    def do_cp_balance_attn(
        self,
        q_nope,
        k_nope,
        q_pe,
        k_pe,
        topk_indices,
        layer,
        actual_seq_qlen,
        actual_seq_lengths_kv,
    ):
        seq_len = q_nope.shape[0]
        split_len = (seq_len + 1) // 2
        q_nope_prev, q_nope_next = torch.split(q_nope, split_len, dim=0)
        q_rope_prev, q_rope_next = torch.split(q_pe, split_len, dim=0)
        q_nope_prev = q_nope_prev.contiguous()
        q_nope_next = q_nope_next.contiguous()
        q_rope_prev = q_rope_prev.contiguous()
        q_rope_next = q_rope_next.contiguous()
        topk_indices_prev, topk_indices_next = topk_indices

        actual_seq_qlen_prev, actual_seq_qlen_next = actual_seq_qlen
        actual_seq_lengths_kv_prev, actual_seq_lengths_kv_next = actual_seq_lengths_kv

        attn_out_prev, _, _ = torch_npu.npu_sparse_flash_attention(
            query=q_nope_prev,
            key=k_nope,
            value=k_nope,
# ... omitted for brevity ...
            layout_kv="PA_BSND",
            sparse_mode=3,
            attention_mode=2,
            return_softmax_lse=False,
        )
        return torch.cat([attn_out_prev, attn_out_next], dim=0)
```
**EN:** This method implements `do_cp_balance_attn` on `AscendAttnBackend`. It primarily calls `torch.split`, `q_nope_prev.contiguous`, `q_nope_next.contiguous`, `q_rope_prev.contiguous`, `q_rope_next.contiguous`, `torch_npu.npu_sparse_flash_attention` to complete its work. State updates are written into `seq_len`, `split_len`, `q_nope_prev`, `q_nope_next`, `q_rope_prev`, `q_rope_next`.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `do_cp_balance_attn`。 它主要通过调用 `torch.split`, `q_nope_prev.contiguous`, `q_nope_next.contiguous`, `q_rope_prev.contiguous`, `q_rope_next.contiguous`, `torch_npu.npu_sparse_flash_attention` 来完成任务。 状态更新主要写入 `seq_len`, `split_len`, `q_nope_prev`, `q_nope_next`, `q_rope_prev`, `q_rope_next`。

### Lines 785-860: Method `AscendAttnBackend.do_cp_attn_fia` / 方法 `AscendAttnBackend.do_cp_attn_fia`
```python
    def do_cp_attn_fia(
        self,
        q: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
        layer: "RadixAttention",
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        """CP-aware attention for standard (non-MLA) models using FIA on Ascend NPU.

        Uses npu_fused_infer_attention_score with paged KV cache (block_table).
        The KV cache must already contain the full gathered sequence
        (written by _cp_allgather_and_save_kv_npu before this call).

        Args:
            q:            Query tensor, shape [total_q_tokens, tp_q_head_num * qk_head_dim]
            k_cache:      Full key cache from token_to_kv_pool
            v_cache:      Full value cache from token_to_kv_pool
            layer:        RadixAttention layer
            forward_batch: ForwardBatch with attn_cp_metadata populated

        Returns:
            attn_output [total_q_tokens, tp_q_head_num * v_head_dim]
        """
        cp_meta = forward_batch.attn_cp_metadata

        # Split Q into prev/next halves per zigzag pattern.
        # torch.chunk(q, 2) gives ceil(n/2) and floor(n/2), matching
# ... omitted for brevity ...
            actual_seq_lengths=[cp_meta.actual_seq_q_next],
            actual_seq_lengths_kv=[cp_meta.kv_len_next],
        )

        attn_out = torch.cat([attn_out_prev, attn_out_next], dim=0)
        return attn_out.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** This method implements `do_cp_attn_fia` on `AscendAttnBackend`. It primarily calls `torch.chunk`, `q_prev.contiguous.reshape`, `q_next.contiguous.reshape`, `k_cache.view`, `v_cache.view`, `torch.ops.npu.npu_fused_infer_attention_score` to complete its work. State updates are written into `cp_meta`, `q_prev`, `q_next`, `k_cache_paged`, `v_cache_paged`, `attn_out`.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `do_cp_attn_fia`。 它主要通过调用 `torch.chunk`, `q_prev.contiguous.reshape`, `q_next.contiguous.reshape`, `k_cache.view`, `v_cache.view`, `torch.ops.npu.npu_fused_infer_attention_score` 来完成任务。 状态更新主要写入 `cp_meta`, `q_prev`, `q_next`, `k_cache_paged`, `v_cache_paged`, `attn_out`。

### Lines 862-972: Method `AscendAttnBackend.forward_sparse` / 方法 `AscendAttnBackend.forward_sparse`
```python
    def forward_sparse(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        # For multi_head latent attention
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: torch.Tensor = None,
    ):

        is_prefill = (
            forward_batch.forward_mode.is_extend()
            and not forward_batch.forward_mode.is_draft_extend_v2()
            and not forward_batch.forward_mode.is_draft_extend()
            and not forward_batch.forward_mode.is_target_verify()
        )

        if save_kv_cache:
            k = k.view(-1, layer.tp_k_head_num, self.kv_lora_rank)
            k_rope = k_rope.view(-1, layer.tp_k_head_num, self.qk_rope_head_dim)
            forward_batch.token_to_kv_pool.set_kv_buffer(
                layer, forward_batch.out_cache_loc, k, k_rope
            )
        q_nope, q_pe = q, q_rope
# ... omitted for brevity ...
                sparse_mode=3,
                attention_mode=2,
                return_softmax_lse=False,
            )

        return attn_out
```
**EN:** This method implements `forward_sparse` on `AscendAttnBackend`. It primarily calls `forward_batch.token_to_kv_pool.get_kv_buffer`, `forward_batch.forward_mode.is_extend`, `k.view`, `k_rope.view`, `forward_batch.token_to_kv_pool.set_kv_buffer`, `is_nsa_enable_prefill_cp` to complete its work. State updates are written into `is_prefill`, `k`, `k_rope`, `actual_seq_lengths_kv`, `attn_out`, `actual_seq_qlen`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `forward_sparse`。 它主要通过调用 `forward_batch.token_to_kv_pool.get_kv_buffer`, `forward_batch.forward_mode.is_extend`, `k.view`, `k_rope.view`, `forward_batch.token_to_kv_pool.set_kv_buffer`, `is_nsa_enable_prefill_cp` 来完成任务。 状态更新主要写入 `is_prefill`, `k`, `k_rope`, `actual_seq_lengths_kv`, `attn_out`, `actual_seq_qlen`。 实现中使用了条件分支。

### Lines 974-1501: Method `AscendAttnBackend.forward_extend` / 方法 `AscendAttnBackend.forward_extend`
```python
    def forward_extend(
        self,
        q,
        k,
        v,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        # For multi_head latent attention
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: Optional[torch.Tensor] = None,
        sinks: Optional[torch.Tensor] = None,
        slopes: Optional[torch.Tensor] = None,
    ):
        if is_mla_preprocess_enabled() and self.use_mla:
            # MLAPO and MLAPROLOG do save kv_cache
            save_kv_cache = False
        if self.is_dllm_model:
            return self.forward_dllm(
                q,
                k,
                v,
                layer,
                forward_batch,
                save_kv_cache,
                q_rope=q_rope,
                k_rope=k_rope,
# ... omitted for brevity ...
                            ),
                        ],
                        dim=0,
                    )

        return attn_output
```
**EN:** This method implements `forward_extend` on `AscendAttnBackend`. It primarily calls `is_mla_preprocess_enabled`, `self.forward_dllm`, `self.forward_sparse`, `forward_batch.forward_mode.is_target_verify`, `forward_batch.forward_mode.is_draft_extend`, `forward_batch.forward_mode.is_draft_extend_v2` to complete its work. State updates are written into `save_kv_cache`, `is_cp_mode`, `k_cache`, `v_cache`, `attn_out`, `q`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `forward_extend`。 它主要通过调用 `is_mla_preprocess_enabled`, `self.forward_dllm`, `self.forward_sparse`, `forward_batch.forward_mode.is_target_verify`, `forward_batch.forward_mode.is_draft_extend`, `forward_batch.forward_mode.is_draft_extend_v2` 来完成任务。 状态更新主要写入 `save_kv_cache`, `is_cp_mode`, `k_cache`, `v_cache`, `attn_out`, `q`。 实现中使用了条件分支、迭代逻辑。

### Lines 1503-1560: Method `AscendAttnBackend.forward_dllm` / 方法 `AscendAttnBackend.forward_dllm`
```python
    def forward_dllm(
        self,
        q,
        k,
        v,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        # For multi_head latent attention
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: Optional[torch.Tensor] = None,
    ):
        if save_kv_cache:
            forward_batch.token_to_kv_pool.set_kv_buffer(
                layer, forward_batch.out_cache_loc, k, v
            )

        k_cache = forward_batch.token_to_kv_pool.get_key_buffer(layer.layer_id)
        v_cache = forward_batch.token_to_kv_pool.get_value_buffer(layer.layer_id)
        query = q.reshape(-1, layer.tp_q_head_num, layer.qk_head_dim)

        if self.forward_metadata.seq_lens_cpu_int is None:
            # capture
            actual_seq_lengths_kv = self.forward_metadata.seq_lens_cpu_list
        else:
            # eagle
            actual_seq_lengths_kv = (
# ... omitted for brevity ...
            actual_seq_lengths=actual_seq_lengths,
            actual_seq_lengths_kv=actual_seq_lengths_kv,
        )
        attn_output = attn_output.view(-1, layer.tp_q_head_num * layer.v_head_dim)

        return attn_output
```
**EN:** This method implements `forward_dllm` on `AscendAttnBackend`. It primarily calls `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `q.reshape`, `torch.ops.npu.npu_fused_infer_attention_score`, `attn_output.view`, `forward_batch.token_to_kv_pool.set_kv_buffer` to complete its work. State updates are written into `k_cache`, `v_cache`, `query`, `attn_output`, `actual_seq_lengths_kv`, `actual_seq_lengths`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `forward_dllm`。 它主要通过调用 `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `q.reshape`, `torch.ops.npu.npu_fused_infer_attention_score`, `attn_output.view`, `forward_batch.token_to_kv_pool.set_kv_buffer` 来完成任务。 状态更新主要写入 `k_cache`, `v_cache`, `query`, `attn_output`, `actual_seq_lengths_kv`, `actual_seq_lengths`。 实现中使用了条件分支。

### Lines 1562-1741: Method `AscendAttnBackend.forward_mtp` / 方法 `AscendAttnBackend.forward_mtp`
```python
    def forward_mtp(
        self,
        q,
        k,
        v,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool,
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
    ):
        if save_kv_cache:
            if self.use_mla:
                k = k.view(-1, layer.tp_k_head_num, self.kv_lora_rank)
                k_rope = k_rope.view(-1, layer.tp_k_head_num, self.qk_rope_head_dim)
                forward_batch.token_to_kv_pool.set_kv_buffer(
                    layer, forward_batch.out_cache_loc, k, k_rope
                )
            else:
                forward_batch.token_to_kv_pool.set_kv_buffer(
                    layer, forward_batch.out_cache_loc, k, v
                )

        if not self.use_mla:
            k_cache = forward_batch.token_to_kv_pool.get_key_buffer(
                layer.layer_id
            ).view(-1, self.page_size, layer.tp_k_head_num * layer.qk_head_dim)
            v_cache = forward_batch.token_to_kv_pool.get_value_buffer(
# ... omitted for brevity ...
                            *attn_output.shape[1:],
                        ),
                    ],
                    dim=0,
                )
            return attn_output
```
**EN:** This method implements `forward_mtp` on `AscendAttnBackend`. It primarily calls `forward_batch.token_to_kv_pool.get_key_buffer.view`, `forward_batch.token_to_kv_pool.get_value_buffer.view`, `q.reshape.contiguous`, `forward_batch.forward_mode.is_draft_extend`, `torch.ops.npu.npu_fused_infer_attention_score`, `attn_output.view` to complete its work. State updates are written into `k_cache`, `v_cache`, `query`, `attn_output`, `q_nope`, `q_rope`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `forward_mtp`。 它主要通过调用 `forward_batch.token_to_kv_pool.get_key_buffer.view`, `forward_batch.token_to_kv_pool.get_value_buffer.view`, `q.reshape.contiguous`, `forward_batch.forward_mode.is_draft_extend`, `torch.ops.npu.npu_fused_infer_attention_score`, `attn_output.view` 来完成任务。 状态更新主要写入 `k_cache`, `v_cache`, `query`, `attn_output`, `q_nope`, `q_rope`。 实现中使用了条件分支。

### Lines 1743-1924: Method `AscendAttnBackend.forward_decode_graph` / 方法 `AscendAttnBackend.forward_decode_graph`
```python
    def forward_decode_graph(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        sinks: Optional[torch.Tensor] = None,
    ):
        if save_kv_cache:
            if self.use_mla:
                k = k.view(-1, layer.tp_k_head_num, self.kv_lora_rank)
                k_rope = k_rope.view(-1, layer.tp_k_head_num, self.qk_rope_head_dim)
                forward_batch.token_to_kv_pool.set_kv_buffer(
                    layer, forward_batch.out_cache_loc, k, k_rope
                )
            else:
                forward_batch.token_to_kv_pool.set_kv_buffer(
                    layer, forward_batch.out_cache_loc, k, v
                )

        if sinks is not None:
            k_cache = forward_batch.token_to_kv_pool.get_key_buffer(layer.layer_id)
            v_cache = forward_batch.token_to_kv_pool.get_value_buffer(layer.layer_id)

# ... omitted for brevity ...
                workspace=workspace,
                out=[output, softmax_lse],
            )

            output = output[:, :, : layer.tp_q_head_num, :]
            return output.view(-1, layer.tp_q_head_num * self.kv_lora_rank)
```
**EN:** This method implements `forward_decode_graph` on `AscendAttnBackend`. It primarily calls `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `attention_sinks_triton`, `forward_batch.token_to_kv_pool.get_key_buffer.view`, `forward_batch.token_to_kv_pool.get_value_buffer.view`, `q.reshape` to complete its work. State updates are written into `k_cache`, `v_cache`, `attn_out`, `query`, `num_tokens`, `workspace`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `forward_decode_graph`。 它主要通过调用 `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `attention_sinks_triton`, `forward_batch.token_to_kv_pool.get_key_buffer.view`, `forward_batch.token_to_kv_pool.get_value_buffer.view`, `q.reshape` 来完成任务。 状态更新主要写入 `k_cache`, `v_cache`, `attn_out`, `query`, `num_tokens`, `workspace`。 实现中使用了条件分支。

### Lines 1926-2195: Method `AscendAttnBackend.forward_decode` / 方法 `AscendAttnBackend.forward_decode`
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        # For multi-head latent attention
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: Optional[torch.Tensor] = None,
        sinks: Optional[torch.Tensor] = None,
        slopes: Optional[torch.Tensor] = None,
    ):
        if is_mla_preprocess_enabled() and self.use_mla:
            # MLAPO does saving kv_cache
            save_kv_cache = False
        if topk_indices is not None:
            return self.forward_sparse(
                q,
                k,
                v,
                layer,
                forward_batch,
                save_kv_cache,
                q_rope,
                k_rope,
# ... omitted for brevity ...
                    block_table=self.forward_metadata.block_tables,
                    context_lens=self.forward_metadata.seq_lens_cpu_int,
                    mla_vheadsize=self.kv_lora_rank,
                    out=attn_output,
                )
            return attn_output.view(num_tokens, layer.tp_q_head_num * self.kv_lora_rank)
```
**EN:** This method implements `forward_decode` on `AscendAttnBackend`. It primarily calls `is_mla_preprocess_enabled`, `self.forward_sparse`, `self.forward_decode_graph`, `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `attn_output.view` to complete its work. State updates are written into `save_kv_cache`, `num_tokens`, `k_cache`, `v_cache`, `kv_c`, `k_pe`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `forward_decode`。 它主要通过调用 `is_mla_preprocess_enabled`, `self.forward_sparse`, `self.forward_decode_graph`, `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `attn_output.view` 来完成任务。 状态更新主要写入 `save_kv_cache`, `num_tokens`, `k_cache`, `v_cache`, `kv_c`, `k_pe`。 实现中使用了条件分支。

### Lines 2197-2250: Method `AscendAttnBackend.forward_mixed` / 方法 `AscendAttnBackend.forward_mixed`
```python
    def forward_mixed(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        topk_indices: Optional[torch.Tensor] = None,
    ):
        if (
            topk_indices is not None
            or self.use_mla
            or (not self.use_fia and layer.qk_head_dim > 128)
        ):
            raise NotImplementedError(
                "The 'enable-mixed-chunk' feature is currently unsupported in the following scenarios: "
                "1. When using the MLA backend on Ascend NPU devices, "
                "2. When using the deepseekv3.2 model on Ascend NPU devices, "
                "3. When the environment variable ASCEND_USE_FIA is set to 0 and qk_head_dim exceeds 128 on Ascend NPU devices."
            )
        if save_kv_cache:
            forward_batch.token_to_kv_pool.set_kv_buffer(
                layer, forward_batch.out_cache_loc, k, v
            )
        k_cache = forward_batch.token_to_kv_pool.get_key_buffer(layer.layer_id)
# ... omitted for brevity ...
            scale=layer.scaling,
        )

        return attn_output.view(
            attn_output.shape[0], layer.tp_q_head_num * layer.v_head_dim
        )
```
**EN:** This method implements `forward_mixed` on `AscendAttnBackend`. It primarily calls `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `k_cache.view`, `v_cache.view`, `q.reshape`, `torch.ops.npu.npu_fused_infer_attention_score` to complete its work. State updates are written into `k_cache`, `v_cache`, `key`, `value`, `query`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendAttnBackend`）实现了 `forward_mixed`。 它主要通过调用 `forward_batch.token_to_kv_pool.get_key_buffer`, `forward_batch.token_to_kv_pool.get_value_buffer`, `k_cache.view`, `v_cache.view`, `q.reshape`, `torch.ops.npu.npu_fused_infer_attention_score` 来完成任务。 状态更新主要写入 `k_cache`, `v_cache`, `key`, `value`, `query`。 实现中使用了条件分支。

### Lines 2253-2257: Class `AscendAttnMultiStepDraftBackend` declaration / 类 `AscendAttnMultiStepDraftBackend` 声明
```python
class AscendAttnMultiStepDraftBackend:
    """
    Wrap multiple Ascend attention backends as one for multiple consecutive
    draft decoding steps
    """
```
**EN:** This class establishes `AscendAttnMultiStepDraftBackend` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `common_template`, `init_forward_metadata`, `init_cuda_graph_state`, `init_forward_metadata_capture_cuda_graph`, `init_forward_metadata_replay_cuda_graph`.
**CN:** 该类将 `AscendAttnMultiStepDraftBackend` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `common_template`, `init_forward_metadata`, `init_cuda_graph_state`, `init_forward_metadata_capture_cuda_graph`, `init_forward_metadata_replay_cuda_graph` 等方法。

### Lines 2259-2272: Method `AscendAttnMultiStepDraftBackend.__init__` / 方法 `AscendAttnMultiStepDraftBackend.__init__`
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        topk: int,
        speculative_num_steps: int,
    ):
        self.topk = topk
        self.speculative_num_steps = speculative_num_steps

        self.attn_backends = []
        for step_id in range(self.speculative_num_steps):
            self.attn_backends.append(
                AscendAttnBackend(model_runner, speculative_step_id=step_id)
            )
```
**EN:** This method implements `__init__` on `AscendAttnMultiStepDraftBackend`. It primarily calls `range`, `self.attn_backends.append`, `AscendAttnBackend` to complete its work. State updates are written into `self.topk`, `self.speculative_num_steps`, `self.attn_backends`. The implementation relies on iteration.
**CN:** 该方法（属于 `AscendAttnMultiStepDraftBackend`）实现了 `__init__`。 它主要通过调用 `range`, `self.attn_backends.append`, `AscendAttnBackend` 来完成任务。 状态更新主要写入 `self.topk`, `self.speculative_num_steps`, `self.attn_backends`。 实现中使用了迭代逻辑。

### Lines 2274-2278: Method `AscendAttnMultiStepDraftBackend.common_template` / 方法 `AscendAttnMultiStepDraftBackend.common_template`
```python
    def common_template(self, forward_batch: ForwardBatch, call_fn: int):
        assert forward_batch.spec_info is not None

        for i in range(self.speculative_num_steps - 1):
            call_fn(i, forward_batch)
```
**EN:** This method implements `common_template` on `AscendAttnMultiStepDraftBackend`. It primarily calls `range`, `call_fn` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `AscendAttnMultiStepDraftBackend`）实现了 `common_template`。 它主要通过调用 `range`, `call_fn` 来完成任务。 实现中使用了迭代逻辑。

### Lines 2280-2285: Method `AscendAttnMultiStepDraftBackend.init_forward_metadata` / 方法 `AscendAttnMultiStepDraftBackend.init_forward_metadata`
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        def call_fn(i, forward_batch):
            assert forward_batch.spec_info is not None
            self.attn_backends[i].init_forward_metadata(forward_batch)

        self.common_template(forward_batch, call_fn)
```
**EN:** This method implements `init_forward_metadata` on `AscendAttnMultiStepDraftBackend`. It primarily calls `self.common_template`, `self.attn_backends.init_forward_metadata` to complete its work.
**CN:** 该方法（属于 `AscendAttnMultiStepDraftBackend`）实现了 `init_forward_metadata`。 它主要通过调用 `self.common_template`, `self.attn_backends.init_forward_metadata` 来完成任务。

### Lines 2287-2289: Method `AscendAttnMultiStepDraftBackend.init_cuda_graph_state` / 方法 `AscendAttnMultiStepDraftBackend.init_cuda_graph_state`
```python
    def init_cuda_graph_state(self, max_bs, max_num_tokens):
        for i in range(self.speculative_num_steps):
            self.attn_backends[i].init_cuda_graph_state(max_bs, max_num_tokens)
```
**EN:** This method implements `init_cuda_graph_state` on `AscendAttnMultiStepDraftBackend`. It primarily calls `range`, `self.attn_backends.init_cuda_graph_state` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `AscendAttnMultiStepDraftBackend`）实现了 `init_cuda_graph_state`。 它主要通过调用 `range`, `self.attn_backends.init_cuda_graph_state` 来完成任务。 实现中使用了迭代逻辑。

### Lines 2291-2303: Method `AscendAttnMultiStepDraftBackend.init_forward_metadata_capture_cuda_graph` / 方法 `AscendAttnMultiStepDraftBackend.init_forward_metadata_capture_cuda_graph`
```python
    def init_forward_metadata_capture_cuda_graph(self, forward_batch: ForwardBatch):
        def call_fn(i, forward_batch):
            self.attn_backends[i].init_forward_metadata_capture_cuda_graph(
                forward_batch.batch_size,
                forward_batch.batch_size * self.topk,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                encoder_lens=None,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
            )

        self.common_template(forward_batch, call_fn)
```
**EN:** This method implements `init_forward_metadata_capture_cuda_graph` on `AscendAttnMultiStepDraftBackend`. It primarily calls `self.common_template`, `self.attn_backends.init_forward_metadata_capture_cuda_graph` to complete its work.
**CN:** 该方法（属于 `AscendAttnMultiStepDraftBackend`）实现了 `init_forward_metadata_capture_cuda_graph`。 它主要通过调用 `self.common_template`, `self.attn_backends.init_forward_metadata_capture_cuda_graph` 来完成任务。

### Lines 2305-2320: Method `AscendAttnMultiStepDraftBackend.init_forward_metadata_replay_cuda_graph` / 方法 `AscendAttnMultiStepDraftBackend.init_forward_metadata_replay_cuda_graph`
```python
    def init_forward_metadata_replay_cuda_graph(
        self, forward_batch: ForwardBatch, bs: int
    ):
        def call_fn(i, forward_batch):
            self.attn_backends[i].init_forward_metadata_replay_cuda_graph(
                bs,
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                seq_lens_sum=-1,
                encoder_lens=None,
                forward_mode=ForwardMode.DECODE,
                spec_info=forward_batch.spec_info,
                seq_lens_cpu=forward_batch.seq_lens_cpu,
            )

        self.common_template(forward_batch, call_fn)
```
**EN:** This method implements `init_forward_metadata_replay_cuda_graph` on `AscendAttnMultiStepDraftBackend`. It primarily calls `self.common_template`, `self.attn_backends.init_forward_metadata_replay_cuda_graph` to complete its work.
**CN:** 该方法（属于 `AscendAttnMultiStepDraftBackend`）实现了 `init_forward_metadata_replay_cuda_graph`。 它主要通过调用 `self.common_template`, `self.attn_backends.init_forward_metadata_replay_cuda_graph` 来完成任务。

## Key Concepts / 关键概念
- **Classes / 类**: `ForwardMetadata`, `AscendAttnMaskBuilder`, `AscendAttnBackend`, `AscendAttnMultiStepDraftBackend`
- **Functions / 函数**: `_reshape_kv_for_fia_nz`, `_cp_allgather_and_save_kv_npu`, `__init__`, `generate_mask_flag`, `generate_attn_mask`, `get_attention_mask_id`, `update_attn_cache`, `get_splitfuse_attn_mask`
- **Themes / 主题**: `attention`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.configs.model_config`, `sglang.srt.dllm.config`, `sglang.srt.hardware_backend.npu.attention.ascend_torch_native_backend`, `sglang.srt.hardware_backend.npu.attention.mla_preprocess`, `sglang.srt.layers.attention.base_attn_backend`, `sglang.srt.layers.attention.nsa.utils`, `sglang.srt.layers.dp_attention`, `sglang.srt.layers.radix_attention`, `sglang.srt.layers.utils.cp_utils`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.speculative.spec_info`, `sglang.srt.utils`
- **External / 外部依赖**: `torch`, `torch_npu`, `sgl_kernel_npu.attention.sinks_attention`, `numpy`
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`, `logging`
