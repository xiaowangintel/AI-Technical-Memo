# lightning_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/linear/lightning_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the lightning backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 lightning backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-17: imports
```python
import logging
import math
from typing import Optional, Union

import torch

from sglang.srt.layers.attention.hybrid_linear_attn_backend import MambaAttnBackendBase
from sglang.srt.layers.attention.linear.lightning_attn import (
    BailingLinearKernel,
    linear_decode_forward_triton,
)
from sglang.srt.layers.attention.linear.linear_metadata import BailingLinearMetadata
from sglang.srt.layers.attention.linear.seg_la import SegLaMeta, seg_la_fwd
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.model_executor.model_runner import ModelRunner
from sglang.srt.speculative.eagle_info import EagleDraftInput, EagleVerifyInput
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 19-19: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 22-38: class LightningAttentionBackend
```python
class LightningAttentionBackend(MambaAttnBackendBase):
    """
    Note about the init:
    - If no spec decoding
        - FlashAttentionBackend will be init once when the server starts.
    - If spec decoding
        - FlashAttentionBackend will be init once for the target worker
        - FlashAttentionMultiStepBackend will be once for the draft worker
            - It will spawn num_steps FlashAttentionBackend for the draft worker

    Note about CUDA Graph:
    - We only support CUDA Graph for Decode (Normal Decode and Draft Decode) and Target Verify.
    - We don't support CUDA Graph for Extend and Draft Extend.
    - When server init, init_cuda_graph_state will be called first and then init_cuda_graph_capture will be called.
    - For each forward batch, init_replay_cuda_graph will be called first and then replay the graph.
    """
```
**EN:** Concrete attention backend that connects lightning attention backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 lightning attention backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 39-68: method LightningAttentionBackend.__init__
```python
    def __init__(self, model_runner: ModelRunner):
        super().__init__(model_runner)

        assert not (
            model_runner.sliding_window_size is not None
            and model_runner.model_config.is_encoder_decoder
        ), "Sliding window and cross attention are not supported together"

        # extra metadata for handling speculative decoding topk > 1, extended draft decode and verify
        self.max_context_len = model_runner.model_config.context_len
        self.device = model_runner.device
        self.decode_cuda_graph_metadata = {}
        self.kv_cache_dtype = model_runner.kv_cache_dtype
        self.kv_cache_dtype_str = model_runner.server_args.kv_cache_dtype
        self.BLOCK = (
            model_runner.model_config.block
            if hasattr(model_runner.model_config, "block")
            else 256
        )
        total_num_heads = model_runner.model_config.hf_config.num_attention_heads
        num_hidden_layers = model_runner.model_config.hf_config.num_hidden_layers
        self.tp_slope = LightningAttentionBackend._build_slope_tensor(
            total_num_heads, num_hidden_layers, self.device
        )
        self.linear_backend = getattr(
            model_runner.model_config.hf_config, "linear_backend", "seg_la"
        )
        logger.info(
            f"linear_backend for linear attention in hybrid_linear_backend: {self.linear_backend}"
        )
```
**EN:** Initializes the LightningAttentionBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 LightningAttentionBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 70-76: method LightningAttentionBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        metadata = self._forward_metadata(forward_batch)
        self.forward_metadata = BailingLinearMetadata.prepare_mixed(
            metadata.query_start_loc,
            metadata.mamba_cache_indices,
            forward_batch,
        )
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 78-91: method LightningAttentionBackend.init_forward_metadata_capture_cuda_graph
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
        metadata = self._capture_metadata(bs, req_pool_indices, forward_mode, spec_info)
        self.forward_metadata = BailingLinearMetadata.prepare_decode(
            metadata.query_start_loc, metadata.mamba_cache_indices, bs, seq_lens
        )
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 93-109: method LightningAttentionBackend.init_forward_metadata_replay_cuda_graph
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
        metadata = self._replay_metadata(
            bs, req_pool_indices, forward_mode, spec_info, seq_lens_cpu
        )
        self.forward_metadata = BailingLinearMetadata.prepare_decode(
            metadata.query_start_loc, metadata.mamba_cache_indices, bs, seq_lens
        )
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 111-155: method LightningAttentionBackend._build_slope_tensor
```python
    @staticmethod
    def _build_slope_tensor(
        n_attention_heads: int, num_hidden_layers: int, device="cuda"
    ):
        def get_slopes(n):
            def get_slopes_power_of_2(n):
                start = 2 ** (-(2 ** -(math.log2(n) - 3)))
                ratio = start
                return [start * ratio**i for i in range(n)]

            if math.log2(n).is_integer():
                return get_slopes_power_of_2(n)
            else:
                closest_power_of_2 = 2 ** math.floor(math.log2(n))
                return (
                    get_slopes_power_of_2(closest_power_of_2)
                    + get_slopes(2 * closest_power_of_2)[0::2][: n - closest_power_of_2]
                )

        slopes = torch.tensor(
            get_slopes(n_attention_heads), dtype=torch.float32
        ).reshape(n_attention_heads, 1, 1)
        from sglang.srt.layers.dp_attention import (
            get_attention_tp_rank,
            get_attention_tp_size,
        )

        tp_heads = n_attention_heads // get_attention_tp_size()
        tp_rank = get_attention_tp_rank()
        if num_hidden_layers <= 1:
            slope_rate_list = [slopes * (1 + 1e-5)]
        else:
            slope_rate_list = [
                slopes * (1 - layer_id / (num_hidden_layers - 1) + 1e-5)
                for layer_id in range(num_hidden_layers)
            ]

        tp_slope = [
            slope_rate_list[layer_id][tp_rank * tp_heads : (tp_rank + 1) * tp_heads]
            .contiguous()
            .to(device)
            for layer_id in range(num_hidden_layers)
        ]

        return tp_slope
```
**EN:** Implements the build slope tensor routine used by this attention module.
**CN:** 实现该注意力模块使用的 build slope tensor 例程。

### Lines 157-216: method LightningAttentionBackend._prefill_and_mix_infer
```python
    def _prefill_and_mix_infer(
        self,
        q,
        k,
        v,
        kv_cache,
        state_indices_tensor,
        forward_batch,
        layer,
        metadata,
    ):
        hidden = []
        for _prefill_idx in range(metadata.num_prefills):
            if _prefill_idx >= forward_batch.extend_start_loc.shape[0]:
                break
            if _prefill_idx >= state_indices_tensor.shape[0]:
                break

            _start = forward_batch.extend_start_loc[_prefill_idx]

            if _prefill_idx + 1 < forward_batch.extend_start_loc.shape[0]:
                _end = forward_batch.extend_start_loc[_prefill_idx + 1]
            else:
                if (
                    forward_batch.extend_seq_lens is not None
                    and _prefill_idx < forward_batch.extend_seq_lens.shape[0]
                    and metadata.num_decodes > 0
                ):
                    seq_len = forward_batch.extend_seq_lens[_prefill_idx]
                    _end = _start + seq_len
                else:
                    _end = q.shape[0]
# ... omitted 16 lines ...
        if metadata.num_decodes > 0:
            hidden.append(
                self._decode_infer(
                    q, k, v, kv_cache, state_indices_tensor, metadata, layer
                )
            )

        if not hidden:
            return torch.empty((0, q.size(-1)), device=q.device, dtype=q.dtype)

        hidden = torch.concat(hidden, dim=0).contiguous()
        return hidden
```
**EN:** Implements the prefill and mix infer routine used by this attention module.
**CN:** 实现该注意力模块使用的 prefill and mix infer 例程。

### Lines 218-233: method LightningAttentionBackend._decode_infer
```python
    def _decode_infer(self, q, k, v, kv_cache, state_indices_tensor, metadata, layer):
        num_prefill_tokens = metadata.num_prefill_tokens
        num_prefills = metadata.num_prefills
        q = q[num_prefill_tokens:].unsqueeze(2).contiguous()
        k = k[num_prefill_tokens:].unsqueeze(2).contiguous()
        v = v[num_prefill_tokens:].unsqueeze(2).contiguous()
        slot_id = state_indices_tensor[num_prefills:]

        assert slot_id.shape[0] == q.shape[0], (
            f"slot_id length {slot_id.shape[0]} does not match decode batch size {q.shape[0]}. "
            "This indicates a bug in the upstream logic that should be investigated."
        )
        hidden = linear_decode_forward_triton(
            q, k, v, kv_cache, self.tp_slope[layer.layer_id], slot_id, 32
        )
        return hidden
```
**EN:** Implements the decode infer routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode infer 例程。

### Lines 235-270: method LightningAttentionBackend._linear_attention_entry
```python
    def _linear_attention_entry(
        self,
        q,
        k,
        v,
        kv_cache,
        state_indices_tensor,
        metadata,
        layer,
        mask=None,
        temp_cache=None,
        intermediate_state_indices=None,
    ):
        q_offsets = metadata.query_start_loc

        seg_meta = SegLaMeta(
            batch_size=metadata.batch_size,
            q_offsets=metadata.query_start_loc,
            s_offsets=state_indices_tensor,
            q_lengths=q_offsets.diff(),
            s_scales=metadata.has_initial_states,
            max_q_length=None,
            mask=mask,
        )
        hidden = seg_la_fwd(
            q=q,
            k=k,
            v=v,
            s=kv_cache,
            decay_scales=self.tp_slope[layer.layer_id],
            meta=seg_meta,
            caches=temp_cache,
            cache_indices=intermediate_state_indices,
            decouple=True,
        )
        return hidden
```
**EN:** Implements the linear attention entry routine used by this attention module.
**CN:** 实现该注意力模块使用的 linear attention entry 例程。

### Lines 272-335: method LightningAttentionBackend.forward_extend
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        **kwargs,
    ):
        q_rope = kwargs["q_rope"] if "q_rope" in kwargs else None
        k_rope = kwargs["k_rope"] if "k_rope" in kwargs else None
        layer_id = layer.layer_id if layer else kwargs["layer_id"]

        metadata = self.forward_metadata

        if self.kv_cache_dtype_str != "auto" and layer.k_scale is not None:
            q = q.to(self.kv_cache_dtype)

        query_start_loc = self.forward_metadata.query_start_loc
        cache_indices = self.forward_metadata.mamba_cache_indices
        mamba_cache_params = self.req_to_token_pool.mamba2_layer_cache(layer_id)
        ssm_states = mamba_cache_params.temporal
        if self.linear_backend == "minimax":
            o = self._prefill_and_mix_infer(
                q.contiguous().view(-1, layer.tp_q_head_num, layer.head_dim),
                k,
                v,
                ssm_states,
                cache_indices,
                forward_batch,
                layer,
# ... omitted 20 lines ...
                temp_cache=(
                    mamba_cache_params.intermediate_ssm
                    if forward_batch.forward_mode.is_target_verify()
                    else None
                ),
                intermediate_state_indices=intermediate_state_indices,
            )
        else:
            raise ValueError(
                f"linear backend: {self.linear_backend} is not support for now"
            )
        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 337-372: method LightningAttentionBackend.forward_decode
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        **kwargs,
    ) -> torch.Tensor:
        q_rope = kwargs["q_rope"] if "q_rope" in kwargs else None
        k_rope = kwargs["k_rope"] if "k_rope" in kwargs else None
        layer_id = layer.layer_id if layer else kwargs["layer_id"]

        # Use precomputed metadata across all layers
        metadata = self.forward_metadata

        if self.kv_cache_dtype_str != "auto":
            q = q.to(self.kv_cache_dtype)

        # Do linear attention
        query_start_loc = self.forward_metadata.query_start_loc
        cache_indices = self.forward_metadata.mamba_cache_indices
        mamba_cache_params = self.req_to_token_pool.mamba2_layer_cache(layer_id)
        ssm_states = mamba_cache_params.temporal
        if self.linear_backend == "minimax":
            o = self._decode_infer(q, k, v, ssm_states, cache_indices, metadata, layer)
        elif self.linear_backend == "seg_la":
            o = self._linear_attention_entry(
                q, k, v, ssm_states, cache_indices, metadata, layer
            )
        else:
            raise ValueError(
                f"linear backend: {self.linear_backend} is not support for now"
            )
        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `logging`
- `math`
- `typing.Optional`
- `typing.Union`
- `torch`
- `sglang.srt.layers.attention.hybrid_linear_attn_backend.MambaAttnBackendBase`
- `sglang.srt.layers.attention.linear.lightning_attn.BailingLinearKernel`
- `sglang.srt.layers.attention.linear.lightning_attn.linear_decode_forward_triton`
- `sglang.srt.layers.attention.linear.linear_metadata.BailingLinearMetadata`
- `sglang.srt.layers.attention.linear.seg_la.SegLaMeta`
- `sglang.srt.layers.attention.linear.seg_la.seg_la_fwd`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `sglang.srt.speculative.eagle_info.EagleDraftInput`
- `sglang.srt.speculative.eagle_info.EagleVerifyInput`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
