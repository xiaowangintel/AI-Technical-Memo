# ascend_gdn_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/attention/ascend_gdn_backend.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Module setup and shared state / 模块设置与共享状态
```python
from typing import Optional, Tuple, Union

import torch
from sgl_kernel_npu.fla.fused_gdn_gating import (
    fused_gdn_gating_kernel_without_sigmoid,
    fused_gdn_gating_npu,
)
from sgl_kernel_npu.mamba.causal_conv1d import (
    causal_conv1d_fn_npu,
    causal_conv1d_update_npu,
    causal_conv1d_update_v2,
)

from sglang.srt.hardware_backend.npu.attention.ascend_hybrid_linear_attn_backend import (
    AscendMambaAttnBackendBase,
)
from sglang.srt.layers.attention.linear.gdn_backend import GDNKernelDispatcher
from sglang.srt.layers.attention.linear.utils import (
    get_linear_attn_decode_backend,
    get_linear_attn_prefill_backend,
)
from sglang.srt.layers.radix_linear_attention import RadixLinearAttention
from sglang.srt.mem_cache.memory_pool import MambaPool
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.model_executor.model_runner import ModelRunner
from sglang.srt.speculative.eagle_info import EagleDraftInput, EagleVerifyInput

fused_gdn_gating = fused_gdn_gating_npu
causal_conv1d_fn = causal_conv1d_fn_npu
causal_conv1d_update = causal_conv1d_update_npu
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `sgl_kernel_npu.fla.fused_gdn_gating`, `sgl_kernel_npu.mamba.causal_conv1d`, `sglang.srt.hardware_backend.npu.attention.ascend_hybrid_linear_attn_backend`, `sglang.srt.layers.attention.linear.gdn_backend`. It also defines symbols such as `fused_gdn_gating`, `causal_conv1d_fn`, `causal_conv1d_update` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `sgl_kernel_npu.fla.fused_gdn_gating`, `sgl_kernel_npu.mamba.causal_conv1d`, `sglang.srt.hardware_backend.npu.attention.ascend_hybrid_linear_attn_backend`, `sglang.srt.layers.attention.linear.gdn_backend`。 同时定义了 `fused_gdn_gating`, `causal_conv1d_fn`, `causal_conv1d_update` 等符号，供后续逻辑使用。

### Lines 33-33: Class `AscendGDNAttnBackend` declaration / 类 `AscendGDNAttnBackend` 声明
```python
class AscendGDNAttnBackend(AscendMambaAttnBackendBase):
```
**EN:** This class establishes `AscendGDNAttnBackend` as the main container/coordinator for the surrounding logic. It inherits from `AscendMambaAttnBackendBase`. Its core interface includes methods such as `__init__`, `prepare_gdn_inputs`, `init_forward_metadata`, `init_forward_metadata_capture_cuda_graph`, `init_forward_metadata_replay_cuda_graph`, `forward_decode`.
**CN:** 该类将 `AscendGDNAttnBackend` 定义为周边逻辑的主要封装体或协调者。 它继承自 `AscendMambaAttnBackendBase`。 其核心接口包括 `__init__`, `prepare_gdn_inputs`, `init_forward_metadata`, `init_forward_metadata_capture_cuda_graph`, `init_forward_metadata_replay_cuda_graph`, `forward_decode` 等方法。

### Lines 35-48: Method `AscendGDNAttnBackend.__init__` / 方法 `AscendGDNAttnBackend.__init__`
```python
    def __init__(self, model_runner: ModelRunner):
        super().__init__(model_runner)
        self.conv_states_shape = torch.Size(
            (
                *model_runner.req_to_token_pool.mamba_pool.mamba_cache.conv[0].shape[
                    :-2
                ],
                model_runner.req_to_token_pool.mamba_pool.mamba_cache.conv[0].shape[-1],
                model_runner.req_to_token_pool.mamba_pool.mamba_cache.conv[0].shape[-2],
            )
        )
        decode_backend = get_linear_attn_decode_backend()
        prefill_backend = get_linear_attn_prefill_backend()
        self.kernel_dispatcher = GDNKernelDispatcher(decode_backend, prefill_backend)
```
**EN:** This method implements `__init__` on `AscendGDNAttnBackend`. It primarily calls `super.__init__`, `torch.Size`, `get_linear_attn_decode_backend`, `get_linear_attn_prefill_backend`, `GDNKernelDispatcher`, `super` to complete its work. State updates are written into `self.conv_states_shape`, `decode_backend`, `prefill_backend`, `self.kernel_dispatcher`.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `torch.Size`, `get_linear_attn_decode_backend`, `get_linear_attn_prefill_backend`, `GDNKernelDispatcher`, `super` 来完成任务。 状态更新主要写入 `self.conv_states_shape`, `decode_backend`, `prefill_backend`, `self.kernel_dispatcher`。

### Lines 50-73: Method `AscendGDNAttnBackend.prepare_gdn_inputs` / 方法 `AscendGDNAttnBackend.prepare_gdn_inputs`
```python
    def prepare_gdn_inputs(
        self,
        bs: int,
        forward_mode: ForwardMode,
        spec_info: Optional[Union[EagleDraftInput, EagleVerifyInput]],
    ):
        cache_indices = self.forward_metadata.mamba_cache_indices
        self.num_accept_tokens = torch.ones(
            [bs], dtype=torch.int32, device=cache_indices.device
        )
        self.actual_seq_lengths = torch.ones(
            [bs], dtype=torch.int32, device=cache_indices.device
        )
        if forward_mode.is_target_verify():
            seq_len = spec_info.draft_token_num
            self.actual_seq_lengths = self.actual_seq_lengths * seq_len
            # indices
            self.ssm_state_indices = torch.arange(
                cache_indices.shape[0] * seq_len,
                dtype=torch.int32,
                device=cache_indices.device,
            )
        else:
            self.ssm_state_indices = cache_indices
```
**EN:** This method implements `prepare_gdn_inputs` on `AscendGDNAttnBackend`. It primarily calls `torch.ones`, `forward_mode.is_target_verify`, `torch.arange` to complete its work. State updates are written into `cache_indices`, `self.num_accept_tokens`, `self.actual_seq_lengths`, `seq_len`, `self.ssm_state_indices`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `prepare_gdn_inputs`。 它主要通过调用 `torch.ones`, `forward_mode.is_target_verify`, `torch.arange` 来完成任务。 状态更新主要写入 `cache_indices`, `self.num_accept_tokens`, `self.actual_seq_lengths`, `seq_len`, `self.ssm_state_indices`。 实现中使用了条件分支。

### Lines 75-84: Method `AscendGDNAttnBackend.init_forward_metadata` / 方法 `AscendGDNAttnBackend.init_forward_metadata`
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        if forward_batch.forward_mode.is_draft_extend(True):
            return
        super().init_forward_metadata(forward_batch)
        self.prepare_gdn_inputs(
            forward_batch.batch_size,
            forward_batch.forward_mode,
            forward_batch.spec_info,
        )
        self.graph_mode = False
```
**EN:** This method implements `init_forward_metadata` on `AscendGDNAttnBackend`. It primarily calls `forward_batch.forward_mode.is_draft_extend`, `super.init_forward_metadata`, `self.prepare_gdn_inputs`, `super` to complete its work. State updates are written into `self.graph_mode`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `init_forward_metadata`。 它主要通过调用 `forward_batch.forward_mode.is_draft_extend`, `super.init_forward_metadata`, `self.prepare_gdn_inputs`, `super` 来完成任务。 状态更新主要写入 `self.graph_mode`。 实现中使用了条件分支。

### Lines 86-108: Method `AscendGDNAttnBackend.init_forward_metadata_capture_cuda_graph` / 方法 `AscendGDNAttnBackend.init_forward_metadata_capture_cuda_graph`
```python
    def init_forward_metadata_capture_cuda_graph(
        self,
        bs: int,
        num_tokens: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[Union[EagleDraftInput, EagleVerifyInput]],
    ):
        if forward_mode.is_draft_extend(True):
            return
        super().init_forward_metadata_capture_cuda_graph(
            bs,
            num_tokens,
            req_pool_indices,
            seq_lens,
            encoder_lens,
            forward_mode,
            spec_info,
        )
        self.prepare_gdn_inputs(bs, forward_mode, spec_info)
        self.graph_mode = True
```
**EN:** This method implements `init_forward_metadata_capture_cuda_graph` on `AscendGDNAttnBackend`. It primarily calls `forward_mode.is_draft_extend`, `super.init_forward_metadata_capture_cuda_graph`, `self.prepare_gdn_inputs`, `super` to complete its work. State updates are written into `self.graph_mode`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `init_forward_metadata_capture_cuda_graph`。 它主要通过调用 `forward_mode.is_draft_extend`, `super.init_forward_metadata_capture_cuda_graph`, `self.prepare_gdn_inputs`, `super` 来完成任务。 状态更新主要写入 `self.graph_mode`。 实现中使用了条件分支。

### Lines 110-134: Method `AscendGDNAttnBackend.init_forward_metadata_replay_cuda_graph` / 方法 `AscendGDNAttnBackend.init_forward_metadata_replay_cuda_graph`
```python
    def init_forward_metadata_replay_cuda_graph(
        self,
        bs: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_sum: int,
        encoder_lens: Optional[torch.Tensor],
        forward_mode: ForwardMode,
        spec_info: Optional[Union[EagleDraftInput, EagleVerifyInput]],
        seq_lens_cpu: Optional[torch.Tensor],
    ):
        if forward_mode.is_draft_extend(True):
            return
        super().init_forward_metadata_replay_cuda_graph(
            bs,
            req_pool_indices,
            seq_lens,
            seq_lens_sum,
            encoder_lens,
            forward_mode,
            spec_info,
            seq_lens_cpu,
        )
        self.prepare_gdn_inputs(bs, forward_mode, spec_info)
        self.graph_mode = True
```
**EN:** This method implements `init_forward_metadata_replay_cuda_graph` on `AscendGDNAttnBackend`. It primarily calls `forward_mode.is_draft_extend`, `super.init_forward_metadata_replay_cuda_graph`, `self.prepare_gdn_inputs`, `super` to complete its work. State updates are written into `self.graph_mode`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `init_forward_metadata_replay_cuda_graph`。 它主要通过调用 `forward_mode.is_draft_extend`, `super.init_forward_metadata_replay_cuda_graph`, `self.prepare_gdn_inputs`, `super` 来完成任务。 状态更新主要写入 `self.graph_mode`。 实现中使用了条件分支。

### Lines 136-189: Method `AscendGDNAttnBackend.forward_decode` / 方法 `AscendGDNAttnBackend.forward_decode`
```python
    def forward_decode(
        self,
        layer: RadixLinearAttention,
        forward_batch: ForwardBatch,
        mixed_qkv: Union[torch.Tensor, Tuple[torch.Tensor, ...]],
        a: torch.Tensor,
        b: torch.Tensor,
        **kwargs,
    ):
        layer_cache = self.req_to_token_pool.mamba2_layer_cache(layer.layer_id)
        conv_states = layer_cache.conv[0]
        ssm_states = layer_cache.temporal
        query_start_loc = self.forward_metadata.query_start_loc
        cache_indices = self.forward_metadata.mamba_cache_indices

        assert isinstance(mixed_qkv, torch.Tensor)
        conv_states_tmp = conv_states.transpose(1, 2).clone()
        mixed_qkv = causal_conv1d_update(
            mixed_qkv,
            conv_states_tmp,
            layer.conv_weights,
            layer.bias,
            layer.activation,
            conv_state_indices=cache_indices,
        )
        conv_states[:] = conv_states_tmp.transpose(1, 2)

        query, key, value = torch.split(
# ... omitted for brevity ...
        )

        self._track_mamba_state_decode(
            forward_batch, conv_states, ssm_states, cache_indices
        )
        return core_attn_out
```
**EN:** This method implements `forward_decode` on `AscendGDNAttnBackend`. It primarily calls `self.req_to_token_pool.mamba2_layer_cache`, `isinstance`, `conv_states.transpose.clone`, `causal_conv1d_update`, `conv_states_tmp.transpose`, `torch.split` to complete its work. State updates are written into `layer_cache`, `conv_states`, `ssm_states`, `query_start_loc`, `cache_indices`, `conv_states_tmp`.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `forward_decode`。 它主要通过调用 `self.req_to_token_pool.mamba2_layer_cache`, `isinstance`, `conv_states.transpose.clone`, `causal_conv1d_update`, `conv_states_tmp.transpose`, `torch.split` 来完成任务。 状态更新主要写入 `layer_cache`, `conv_states`, `ssm_states`, `query_start_loc`, `cache_indices`, `conv_states_tmp`。

### Lines 191-367: Method `AscendGDNAttnBackend.forward_extend` / 方法 `AscendGDNAttnBackend.forward_extend`
```python
    def forward_extend(
        self,
        layer: RadixLinearAttention,
        forward_batch: ForwardBatch,
        mixed_qkv: Union[torch.Tensor, Tuple[torch.Tensor, ...]],
        a: torch.Tensor,
        b: torch.Tensor,
        **kwargs,
    ):
        assert isinstance(mixed_qkv, torch.Tensor)
        seq_len = mixed_qkv.shape[0]
        is_target_verify = forward_batch.forward_mode.is_target_verify()
        forward_metadata = self.forward_metadata

        query_start_loc = forward_metadata.query_start_loc
        cache_indices = forward_metadata.mamba_cache_indices
        retrieve_next_token = forward_metadata.retrieve_next_token
        retrieve_next_sibling = forward_metadata.retrieve_next_sibling
        retrieve_parent_token = forward_metadata.retrieve_parent_token

        mamba_cache_params = self.req_to_token_pool.mamba2_layer_cache(layer.layer_id)
        conv_states = mamba_cache_params.conv[0]
        ssm_states = mamba_cache_params.temporal
        if is_target_verify:
            assert isinstance(mamba_cache_params, MambaPool.SpeculativeState)
            intermediate_state_cache = mamba_cache_params.intermediate_ssm
            intermediate_conv_window_cache = (
                mamba_cache_params.intermediate_conv_window[0]
# ... omitted for brevity ...
            if h is not None:
                self._track_mamba_state_extend(
                    forward_batch, h, ssm_states, forward_metadata
                )

        return core_attn_out
```
**EN:** This method implements `forward_extend` on `AscendGDNAttnBackend`. It primarily calls `isinstance`, `forward_batch.forward_mode.is_target_verify`, `self.req_to_token_pool.mamba2_layer_cache`, `torch.ones`, `torch.full`, `causal_conv1d_update_v2.view` to complete its work. State updates are written into `seq_len`, `is_target_verify`, `forward_metadata`, `query_start_loc`, `cache_indices`, `retrieve_next_token`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `forward_extend`。 它主要通过调用 `isinstance`, `forward_batch.forward_mode.is_target_verify`, `self.req_to_token_pool.mamba2_layer_cache`, `torch.ones`, `torch.full`, `causal_conv1d_update_v2.view` 来完成任务。 状态更新主要写入 `seq_len`, `is_target_verify`, `forward_metadata`, `query_start_loc`, `cache_indices`, `retrieve_next_token`。 实现中使用了条件分支。

### Lines 369-425: Method `AscendGDNAttnBackend.fused_recurrent_gated_delta_rule_update` / 方法 `AscendGDNAttnBackend.fused_recurrent_gated_delta_rule_update`
```python
    def fused_recurrent_gated_delta_rule_update(
        self,
        mix_qkv: torch.Tensor,
        num_heads,
        num_value_heads,
        head_k_dim,
        head_v_dim,
        recurrent_state: torch.Tensor,
        beta: torch.Tensor,
        g: torch.Tensor,
        cache_indices: torch.Tensor,
        intermediate_state: Optional[torch.Tensor] = None,
    ):
        beta = beta.to(torch.bfloat16)
        g = g.to(torch.float32)
        batch_size = mix_qkv.shape[0]
        seq_len = mix_qkv.shape[1]
        scale = 1 / (head_k_dim**0.5)

        if intermediate_state is not None:
            intermediate_state = intermediate_state.view(
                -1, num_value_heads, head_k_dim, head_v_dim
            )

        if self.graph_mode:
            num_accept_tokens = torch.full(
                [batch_size], 1, dtype=torch.int32, device=cache_indices.device
            )
# ... omitted for brevity ...

        if intermediate_state is not None:
            intermediate_state = intermediate_state.view(
                -1, seq_len, num_value_heads, head_k_dim, head_v_dim
            )
        return attn_core_out
```
**EN:** This method implements `fused_recurrent_gated_delta_rule_update` on `AscendGDNAttnBackend`. It primarily calls `beta.to`, `g.to`, `torch.ops.npu.recurrent_gated_delta_rule`, `intermediate_state.view`, `torch.full`, `ssm_state_indices.view` to complete its work. State updates are written into `beta`, `g`, `batch_size`, `seq_len`, `scale`, `attn_core_out`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendGDNAttnBackend`）实现了 `fused_recurrent_gated_delta_rule_update`。 它主要通过调用 `beta.to`, `g.to`, `torch.ops.npu.recurrent_gated_delta_rule`, `intermediate_state.view`, `torch.full`, `ssm_state_indices.view` 来完成任务。 状态更新主要写入 `beta`, `g`, `batch_size`, `seq_len`, `scale`, `attn_core_out`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `AscendGDNAttnBackend`
- **Functions / 函数**: `__init__`, `prepare_gdn_inputs`, `init_forward_metadata`, `init_forward_metadata_capture_cuda_graph`, `init_forward_metadata_replay_cuda_graph`, `forward_decode`, `forward_extend`, `fused_recurrent_gated_delta_rule_update`
- **Themes / 主题**: `attention`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.npu.attention.ascend_hybrid_linear_attn_backend`, `sglang.srt.layers.attention.linear.gdn_backend`, `sglang.srt.layers.attention.linear.utils`, `sglang.srt.layers.radix_linear_attention`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.model_executor.model_runner`, `sglang.srt.speculative.eagle_info`
- **External / 外部依赖**: `torch`, `sgl_kernel_npu.fla.fused_gdn_gating`, `sgl_kernel_npu.mamba.causal_conv1d`
- **Standard library / 标准库**: `typing`
