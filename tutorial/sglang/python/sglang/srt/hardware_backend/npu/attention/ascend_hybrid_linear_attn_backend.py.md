# ascend_hybrid_linear_attn_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/attention/ascend_hybrid_linear_attn_backend.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Module setup and shared state / 模块设置与共享状态
```python
import logging
from typing import Optional, Union

import torch
from sgl_kernel_npu.mamba.mamba_state_update_triton import (
    conv_state_rollback,
    move_intermediate_cache,
)

from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.attention.hybrid_linear_attn_backend import (
    HybridLinearAttnBackend,
    MambaAttnBackendBase,
)
from sglang.srt.layers.attention.mamba.mamba2_metadata import (
    ForwardMetadata,
)
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.model_executor.model_runner import ModelRunner
from sglang.srt.speculative.eagle_info import EagleDraftInput, EagleVerifyInput
from sglang.srt.speculative.spec_info import SpecInput

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `typing`, `torch`, `sgl_kernel_npu.mamba.mamba_state_update_triton`, `sglang.srt.layers.attention.base_attn_backend`, `sglang.srt.layers.attention.hybrid_linear_attn_backend`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `typing`, `torch`, `sgl_kernel_npu.mamba.mamba_state_update_triton`, `sglang.srt.layers.attention.base_attn_backend`, `sglang.srt.layers.attention.hybrid_linear_attn_backend`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 26-26: Class `AscendMambaAttnBackendBase` declaration / 类 `AscendMambaAttnBackendBase` 声明
```python
class AscendMambaAttnBackendBase(MambaAttnBackendBase):
```
**EN:** This class establishes `AscendMambaAttnBackendBase` as the main container/coordinator for the surrounding logic. It inherits from `MambaAttnBackendBase`. Its core interface includes methods such as `__init__`, `init_cuda_graph_state`, `_capture_metadata`, `_replay_metadata`, `get_cuda_graph_seq_len_fill_value`.
**CN:** 该类将 `AscendMambaAttnBackendBase` 定义为周边逻辑的主要封装体或协调者。 它继承自 `MambaAttnBackendBase`。 其核心接口包括 `__init__`, `init_cuda_graph_state`, `_capture_metadata`, `_replay_metadata`, `get_cuda_graph_seq_len_fill_value` 等方法。

### Lines 27-29: Method `AscendMambaAttnBackendBase.__init__` / 方法 `AscendMambaAttnBackendBase.__init__`
```python
    def __init__(self, model_runner: ModelRunner):
        super().__init__(model_runner)
        self.state_indices_list_gdn = []
```
**EN:** This method implements `__init__` on `AscendMambaAttnBackendBase`. It primarily calls `super.__init__`, `super` to complete its work. State updates are written into `self.state_indices_list_gdn`.
**CN:** 该方法（属于 `AscendMambaAttnBackendBase`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self.state_indices_list_gdn`。

### Lines 31-77: Method `AscendMambaAttnBackendBase.init_cuda_graph_state` / 方法 `AscendMambaAttnBackendBase.init_cuda_graph_state`
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        assert (
            max_num_tokens % max_bs == 0
        ), f"max_num_tokens={max_num_tokens} must be divisible by max_bs={max_bs}"
        draft_token_num = max_num_tokens // max_bs
        for i in range(max_bs):
            self.state_indices_list.append(
                torch.full(
                    (i + 1,), self.pad_slot_id, dtype=torch.int32, device=self.device
                )
            )
            self.state_indices_list_gdn.append(
                torch.full(
                    ((i + 1) * draft_token_num,),
                    self.pad_slot_id,
                    dtype=torch.int32,
                    device=self.device,
                )
            )
            self.query_start_loc_list.append(
                torch.zeros((i + 2,), dtype=torch.int32, device=self.device)
            )
            self.retrieve_next_token_list.append(
                torch.zeros(
                    (i + 1, draft_token_num), dtype=torch.int32, device=self.device
                )
            )
            self.retrieve_next_sibling_list.append(
# ... omitted for brevity ...
            0,
            max_bs * draft_token_num + 1,
            step=draft_token_num,
            dtype=torch.int32,
            device=self.device,
        )
```
**EN:** This method implements `init_cuda_graph_state` on `AscendMambaAttnBackendBase`. It primarily calls `range`, `torch.arange`, `self.state_indices_list.append`, `self.state_indices_list_gdn.append`, `self.query_start_loc_list.append`, `self.retrieve_next_token_list.append` to complete its work. State updates are written into `draft_token_num`, `self.cached_cuda_graph_decode_query_start_loc`, `self.cached_cuda_graph_verify_query_start_loc`. The implementation relies on iteration.
**CN:** 该方法（属于 `AscendMambaAttnBackendBase`）实现了 `init_cuda_graph_state`。 它主要通过调用 `range`, `torch.arange`, `self.state_indices_list.append`, `self.state_indices_list_gdn.append`, `self.query_start_loc_list.append`, `self.retrieve_next_token_list.append` 来完成任务。 状态更新主要写入 `draft_token_num`, `self.cached_cuda_graph_decode_query_start_loc`, `self.cached_cuda_graph_verify_query_start_loc`。 实现中使用了迭代逻辑。

### Lines 79-124: Method `AscendMambaAttnBackendBase._capture_metadata` / 方法 `AscendMambaAttnBackendBase._capture_metadata`
```python
    def _capture_metadata(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        forward_mode: ForwardMode,
        spec_info: Optional[Union[EagleDraftInput, EagleVerifyInput]],
    ):
        mamba_indices = self.req_to_token_pool.get_mamba_indices(req_pool_indices)
        self.state_indices_list[bs - 1][: len(mamba_indices)].copy_(mamba_indices)
        if forward_mode.is_decode_or_idle():
            self.query_start_loc_list[bs - 1].copy_(
                self.cached_cuda_graph_decode_query_start_loc[: bs + 1]
            )
        elif forward_mode.is_target_verify():
            self.query_start_loc_list[bs - 1].copy_(
                self.cached_cuda_graph_verify_query_start_loc[: bs + 1]
            )
            ssm_state_indices = torch.arange(
                mamba_indices.shape[0] * spec_info.draft_token_num,
                dtype=torch.int32,
                device=mamba_indices.device,
            )
            self.state_indices_list_gdn[bs - 1][
                : len(mamba_indices) * spec_info.draft_token_num
            ].copy_(ssm_state_indices)
        else:
            raise ValueError(f"Invalid forward mode: {forward_mode=}")

# ... omitted for brevity ...
        else:
            return ForwardMetadata(
                query_start_loc=self.query_start_loc_list[bs - 1],
                mamba_cache_indices=self.state_indices_list[bs - 1],
                mamba_cache_indices_gdn=self.state_indices_list_gdn[bs - 1],
            )
```
**EN:** This method implements `_capture_metadata` on `AscendMambaAttnBackendBase`. It primarily calls `self.req_to_token_pool.get_mamba_indices`, `self.state_indices_list.copy_`, `forward_mode.is_decode_or_idle`, `self.query_start_loc_list.copy_`, `forward_mode.is_target_verify`, `ForwardMetadata` to complete its work. State updates are written into `mamba_indices`, `ssm_state_indices`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendMambaAttnBackendBase`）实现了 `_capture_metadata`。 它主要通过调用 `self.req_to_token_pool.get_mamba_indices`, `self.state_indices_list.copy_`, `forward_mode.is_decode_or_idle`, `self.query_start_loc_list.copy_`, `forward_mode.is_target_verify`, `ForwardMetadata` 来完成任务。 状态更新主要写入 `mamba_indices`, `ssm_state_indices`。 实现中使用了条件分支。

### Lines 126-201: Method `AscendMambaAttnBackendBase._replay_metadata` / 方法 `AscendMambaAttnBackendBase._replay_metadata`
```python
    def _replay_metadata(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        forward_mode: ForwardMode,
        spec_info: Optional[SpecInput],
        seq_lens_cpu: Optional[torch.Tensor],
    ):
        num_padding = torch.count_nonzero(
            seq_lens_cpu == self.get_cuda_graph_seq_len_fill_value()
        )
        # Make sure forward metadata is correctly handled for padding reqs
        req_pool_indices[bs - num_padding :] = 0
        mamba_indices = self.req_to_token_pool.get_mamba_indices(req_pool_indices)
        mamba_indices[bs - num_padding :] = 0
        self.state_indices_list[bs - 1][: len(mamba_indices)].copy_(mamba_indices)
        if forward_mode.is_decode_or_idle():
            if num_padding == 0:
                self.query_start_loc_list[bs - 1].copy_(
                    self.cached_cuda_graph_decode_query_start_loc[: bs + 1]
                )
            else:
                self.query_start_loc_list[bs - 1][: bs - num_padding].copy_(
                    self.cached_cuda_graph_decode_query_start_loc[: bs - num_padding]
                )
                self.query_start_loc_list[bs - 1][bs - num_padding :].fill_(
                    bs - num_padding
                )
# ... omitted for brevity ...
        else:
            return ForwardMetadata(
                query_start_loc=self.query_start_loc_list[bs - 1],
                mamba_cache_indices=self.state_indices_list[bs - 1],
                mamba_cache_indices_gdn=self.state_indices_list_gdn[bs - 1],
            )
```
**EN:** This method implements `_replay_metadata` on `AscendMambaAttnBackendBase`. It primarily calls `torch.count_nonzero`, `self.req_to_token_pool.get_mamba_indices`, `self.state_indices_list.copy_`, `forward_mode.is_decode_or_idle`, `forward_mode.is_target_verify`, `self.retrieve_next_token_list.copy_` to complete its work. State updates are written into `num_padding`, `req_pool_indices`, `mamba_indices`, `bs_without_pad`, `ssm_state_indices`, `self.state_indices_list_gdn`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendMambaAttnBackendBase`）实现了 `_replay_metadata`。 它主要通过调用 `torch.count_nonzero`, `self.req_to_token_pool.get_mamba_indices`, `self.state_indices_list.copy_`, `forward_mode.is_decode_or_idle`, `forward_mode.is_target_verify`, `self.retrieve_next_token_list.copy_` 来完成任务。 状态更新主要写入 `num_padding`, `req_pool_indices`, `mamba_indices`, `bs_without_pad`, `ssm_state_indices`, `self.state_indices_list_gdn`。 实现中使用了条件分支。

### Lines 203-204: Method `AscendMambaAttnBackendBase.get_cuda_graph_seq_len_fill_value` / 方法 `AscendMambaAttnBackendBase.get_cuda_graph_seq_len_fill_value`
```python
    def get_cuda_graph_seq_len_fill_value(self):
        return 0  # Mamba attn does not use seq lens to index kv cache
```
**EN:** This method implements `get_cuda_graph_seq_len_fill_value` on `AscendMambaAttnBackendBase`.
**CN:** 该方法（属于 `AscendMambaAttnBackendBase`）实现了 `get_cuda_graph_seq_len_fill_value`。

### Lines 207-208: Class `AscendMamba2AttnBackend` declaration / 类 `AscendMamba2AttnBackend` 声明
```python
class AscendMamba2AttnBackend(AscendMambaAttnBackendBase):
    pass
```
**EN:** This class establishes `AscendMamba2AttnBackend` as the main container/coordinator for the surrounding logic. It inherits from `AscendMambaAttnBackendBase`.
**CN:** 该类将 `AscendMamba2AttnBackend` 定义为周边逻辑的主要封装体或协调者。 它继承自 `AscendMambaAttnBackendBase`。

### Lines 211-211: Class `AscendHybridLinearAttnBackend` declaration / 类 `AscendHybridLinearAttnBackend` 声明
```python
class AscendHybridLinearAttnBackend(HybridLinearAttnBackend):
```
**EN:** This class establishes `AscendHybridLinearAttnBackend` as the main container/coordinator for the surrounding logic. It inherits from `HybridLinearAttnBackend`. Its core interface includes methods such as `__init__`, `update_mamba_state_after_mtp_verify`, `update_verify_buffers_to_fill_after_draft`.
**CN:** 该类将 `AscendHybridLinearAttnBackend` 定义为周边逻辑的主要封装体或协调者。 它继承自 `HybridLinearAttnBackend`。 其核心接口包括 `__init__`, `update_mamba_state_after_mtp_verify`, `update_verify_buffers_to_fill_after_draft` 等方法。

### Lines 212-218: Method `AscendHybridLinearAttnBackend.__init__` / 方法 `AscendHybridLinearAttnBackend.__init__`
```python
    def __init__(
        self,
        full_attn_backend: AttentionBackend,
        linear_attn_backend: AscendMambaAttnBackendBase,
        full_attn_layers: list[int],
    ):
        super().__init__(full_attn_backend, linear_attn_backend, full_attn_layers)
```
**EN:** This method implements `__init__` on `AscendHybridLinearAttnBackend`. It primarily calls `super.__init__`, `super` to complete its work.
**CN:** 该方法（属于 `AscendHybridLinearAttnBackend`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。

### Lines 220-275: Method `AscendHybridLinearAttnBackend.update_mamba_state_after_mtp_verify` / 方法 `AscendHybridLinearAttnBackend.update_mamba_state_after_mtp_verify`
```python
    def update_mamba_state_after_mtp_verify(
        self,
        last_correct_step_indices: torch.Tensor,
        mamba_track_indices: Optional[torch.Tensor],
        mamba_steps_to_track: Optional[torch.Tensor],
        model,
    ):
        """
        Update mamba states after MTP verify using fully fused Triton kernel.

        This replaces the original advanced indexing operations with a single fused
        gather-scatter kernel that also handles masking internally, avoiding:
        - index_elementwise_kernel from tensor[bool_mask]
        - index_select kernel launches
        - nonzero kernel launches
        """
        request_number = last_correct_step_indices.shape[0]

        state_indices_tensor = (
            self.linear_attn_backend.forward_metadata.mamba_cache_indices[
                :request_number
            ]
        )

        mamba_caches = (
            self.linear_attn_backend.req_to_token_pool.get_speculative_mamba2_params_all_layers()
        )

# ... omitted for brevity ...
                conv_states,
                dst_indices_tensor,
                last_steps,
                draft_token_num,
            )
        return
```
**EN:** This method implements `update_mamba_state_after_mtp_verify` on `AscendHybridLinearAttnBackend`. It primarily calls `self.linear_attn_backend.req_to_token_pool.get_speculative_mamba2_params_all_layers`, `state_indices_tensor.to`, `torch.arange`, `last_correct_step_indices.to`, `move_intermediate_cache`, `dst_indices_tensor.numel` to complete its work. State updates are written into `request_number`, `state_indices_tensor`, `mamba_caches`, `conv_states`, `ssm_states`, `intermediate_state_cache`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendHybridLinearAttnBackend`）实现了 `update_mamba_state_after_mtp_verify`。 它主要通过调用 `self.linear_attn_backend.req_to_token_pool.get_speculative_mamba2_params_all_layers`, `state_indices_tensor.to`, `torch.arange`, `last_correct_step_indices.to`, `move_intermediate_cache`, `dst_indices_tensor.numel` 来完成任务。 状态更新主要写入 `request_number`, `state_indices_tensor`, `mamba_caches`, `conv_states`, `ssm_states`, `intermediate_state_cache`。 实现中使用了条件分支。

### Lines 277-280: Method `AscendHybridLinearAttnBackend.update_verify_buffers_to_fill_after_draft` / 方法 `AscendHybridLinearAttnBackend.update_verify_buffers_to_fill_after_draft`
```python
    def update_verify_buffers_to_fill_after_draft(
        self, spec_info: SpecInput, cuda_graph_bs: Optional[int]
    ):
        pass
```
**EN:** This method implements `update_verify_buffers_to_fill_after_draft` on `AscendHybridLinearAttnBackend`.
**CN:** 该方法（属于 `AscendHybridLinearAttnBackend`）实现了 `update_verify_buffers_to_fill_after_draft`。

## Key Concepts / 关键概念
- **Classes / 类**: `AscendMambaAttnBackendBase`, `AscendMamba2AttnBackend`, `AscendHybridLinearAttnBackend`
- **Functions / 函数**: `__init__`, `init_cuda_graph_state`, `_capture_metadata`, `_replay_metadata`, `get_cuda_graph_seq_len_fill_value`, `__init__`, `update_mamba_state_after_mtp_verify`, `update_verify_buffers_to_fill_after_draft`
- **Themes / 主题**: `attention`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.layers.attention.base_attn_backend`, `sglang.srt.layers.attention.hybrid_linear_attn_backend`, `sglang.srt.layers.attention.mamba.mamba2_metadata`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.model_executor.model_runner`, `sglang.srt.speculative.eagle_info`, `sglang.srt.speculative.spec_info`
- **External / 外部依赖**: `torch`, `sgl_kernel_npu.mamba.mamba_state_update_triton`
- **Standard library / 标准库**: `logging`, `typing`
