# torch_flex_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/torch_flex_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the torch flex backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 torch flex backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-10: imports
```python
from __future__ import annotations

from typing import TYPE_CHECKING

import torch
from torch.nn.attention.flex_attention import create_block_mask, flex_attention

from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.radix_attention import AttentionType
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 12-14: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 17-17: class TorchFlexAttnBackend
```python
class TorchFlexAttnBackend(AttentionBackend):
```
**EN:** Concrete attention backend that connects torch flex attn backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 torch flex attn backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 18-24: method TorchFlexAttnBackend.__init__
```python
    def __init__(self, model_runner: ModelRunner):
        super().__init__()
        self.forward_metadata = None
        self.device = model_runner.device
        self.flex_attention = torch.compile(flex_attention, dynamic=True)
        torch._dynamo.config.cache_size_limit = 1024
        torch._dynamo.config.accumulated_cache_size_limit = 1024
```
**EN:** Initializes the TorchFlexAttnBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TorchFlexAttnBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 26-67: method TorchFlexAttnBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        """Init the metadata for a forward pass."""
        # TODO: find a more elegant way to save memory
        # Currently maintain the same memory as torch_native_backend
        torch.cuda.empty_cache()

        # Provide two block_mask Lists per seq_idx for lower latency, later will support per layer level mask generation
        self.extend_block_masks = []
        self.decode_block_masks = []

        if forward_batch.forward_mode.is_extend():
            for seq_idx in range(forward_batch.seq_lens.shape[0]):
                seq_len_kv = forward_batch.seq_lens[seq_idx]
                seq_len_q = seq_len_kv
                self.extend_block_masks.append(
                    create_block_mask(
                        self._causal_mask,
                        None,
                        None,
                        seq_len_q,
                        seq_len_kv,
                        device=self.device,
                        _compile=False,
                    )
                )

        elif forward_batch.forward_mode.is_decode():
            for seq_idx in range(forward_batch.seq_lens.shape[0]):
                seq_len_q = 1
                seq_len_kv = forward_batch.seq_lens[seq_idx]

                self.decode_block_masks.append(
                    create_block_mask(
                        self._decode_mask,
                        None,
                        None,
                        seq_len_q,
                        seq_len_kv,
                        device=self.device,
                        _compile=False,
                    )
                )
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 69-70: method TorchFlexAttnBackend._causal_mask
```python
    def _causal_mask(self, b, h, q_idx, kv_idx):
        return q_idx >= kv_idx
```
**EN:** Implements the causal mask routine used by this attention module.
**CN:** 实现该注意力模块使用的 causal mask 例程。

### Lines 72-73: method TorchFlexAttnBackend._decode_mask
```python
    def _decode_mask(self, b, h, q_idx, kv_idx):
        return q_idx <= kv_idx
```
**EN:** Implements the decode mask routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode mask 例程。

### Lines 75-163: method TorchFlexAttnBackend._run_flex_forward_extend
```python
    def _run_flex_forward_extend(
        self,
        query: torch.Tensor,
        output: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
        req_to_token: torch.Tensor,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        extend_prefix_lens: torch.Tensor,
        extend_seq_lens: torch.Tensor,
        scaling=None,
        enable_gqa=False,
        causal=False,
    ):
        """Run the extend forward by using torch flex attention op.

        Args:
            query: [num_tokens, num_heads, head_size]
            output: [num_tokens, num_heads, head_size]
            k_cache: [max_total_num_tokens, num_heads, head_size]
            v_cache: [max_total_num_tokens, num_heads, head_size]
            req_to_token: [max_num_reqs, max_context_len]
            req_pool_indices: [num_seqs]
            seq_lens: [num_seqs]
            extend_prefix_lens: [num_seqs]
            extend_seq_lens: [num_seqs]
            scaling: float or None
            enable_gqa: bool
            causal: bool

        Returns:
# ... omitted 45 lines ...
                    block_mask=self.extend_block_masks[seq_idx],
                    scale=scaling,
                    enable_gqa=enable_gqa,
                )
                .squeeze(0)
                .movedim(query.dim() - 2, 0)
            )
            output[start_q:end_q, :, :] = per_req_out_redundant[
                prefill_seq_len_q:, :, :
            ]
            start_q, start_kv = end_q, end_kv
        return output
```
**EN:** Runs the forward-path logic for run flex forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 run flex forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 165-234: method TorchFlexAttnBackend._run_flex_forward_decode
```python
    def _run_flex_forward_decode(
        self,
        query: torch.Tensor,
        output: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
        req_to_token: torch.Tensor,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        scaling=None,
        enable_gqa=False,
        causal=False,
    ):
        """Run the decode forward by using torch flex attention op.

        Args:
            query: [num_tokens, num_heads, head_size]
            output: [num_tokens, num_heads, head_size]
            k_cache: [max_total_num_tokens, num_heads, head_size]
            v_cache: [max_total_num_tokens, num_heads, head_size]
            req_to_token: [max_num_reqs, max_context_len]
            req_pool_indices: [num_seqs]
            seq_lens: [num_seqs]
            scaling: float or None
            enable_gqa: bool
            causal: bool

        Returns:
            output: [num_tokens, num_heads, head_size]
        """

        # [num_tokens, num_heads, head_size] -> [num_heads, num_tokens, head_size]
# ... omitted 26 lines ...
                    block_mask=self.decode_block_masks[seq_idx],
                    scale=scaling,
                    enable_gqa=enable_gqa,
                )
                .squeeze(0)
                .movedim(query.dim() - 2, 0)
            )

            output[start_q:end_q, :, :] = per_req_out
            start_q, start_kv = end_q, end_kv

        return output
```
**EN:** Runs the forward-path logic for run flex forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 run flex forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 236-280: method TorchFlexAttnBackend.forward_extend
```python
    def forward_extend(
        self,
        q,
        k,
        v,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
    ):
        if layer.qk_head_dim != layer.v_head_dim:
            o = q.new_empty((q.shape[0], layer.tp_q_head_num * layer.v_head_dim))
        else:
            o = torch.empty_like(q)

        if save_kv_cache:
            forward_batch.token_to_kv_pool.set_kv_buffer(
                layer, forward_batch.out_cache_loc, k, v
            )

        use_gqa = layer.tp_q_head_num != layer.tp_k_head_num

        q_ = q.view(-1, layer.tp_q_head_num, layer.qk_head_dim)
        o_ = o.view(-1, layer.tp_q_head_num, layer.v_head_dim)

        causal = True
        if layer.is_cross_attention or layer.attn_type == AttentionType.ENCODER_ONLY:
            raise NotImplementedError(
                "TorchFlexAttnBackend does not support non-causal attention for now."
            )

        self._run_flex_forward_extend(
            q_,
            o_,
            forward_batch.token_to_kv_pool.get_key_buffer(layer.layer_id),
            forward_batch.token_to_kv_pool.get_value_buffer(layer.layer_id),
            forward_batch.req_to_token_pool.req_to_token,
            forward_batch.req_pool_indices,
            forward_batch.seq_lens,
            forward_batch.extend_prefix_lens,
            forward_batch.extend_seq_lens,
            scaling=layer.scaling,
            enable_gqa=use_gqa,
            causal=causal,
        )
        return o
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 282-322: method TorchFlexAttnBackend.forward_decode
```python
    def forward_decode(
        self,
        q,
        k,
        v,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
    ):
        # During torch.compile, there is a bug in rotary_emb that causes the
        # output value to have a 3D tensor shape. This reshapes the output correctly.
        q = q.reshape(-1, layer.tp_q_head_num * layer.qk_head_dim)

        if layer.qk_head_dim != layer.v_head_dim:
            o = q.new_empty((q.shape[0], layer.tp_q_head_num * layer.v_head_dim))
        else:
            o = torch.empty_like(q)

        if save_kv_cache:
            forward_batch.token_to_kv_pool.set_kv_buffer(
                layer, forward_batch.out_cache_loc, k, v
            )

        use_gqa = layer.tp_q_head_num != layer.tp_k_head_num
        q_ = q.view(-1, layer.tp_q_head_num, layer.qk_head_dim)
        o_ = o.view(-1, layer.tp_q_head_num, layer.v_head_dim)

        self._run_flex_forward_decode(
            q_,
            o_,
            forward_batch.token_to_kv_pool.get_key_buffer(layer.layer_id),
            forward_batch.token_to_kv_pool.get_value_buffer(layer.layer_id),
            forward_batch.req_to_token_pool.req_to_token,
            forward_batch.req_pool_indices,
            forward_batch.seq_lens,
            scaling=layer.scaling,
            enable_gqa=use_gqa,
            causal=False,
        )

        return o
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 324-325: method TorchFlexAttnBackend.support_triton
```python
    def support_triton(self):
        return False
```
**EN:** Implements the support triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 support triton 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `__future__.annotations`
- `typing.TYPE_CHECKING`
- `torch`
- `torch.nn.attention.flex_attention.create_block_mask`
- `torch.nn.attention.flex_attention.flex_attention`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.layers.radix_attention.AttentionType`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
