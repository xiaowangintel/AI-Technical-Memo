# hybrid_attn_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/hybrid_attn_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the hybrid attn backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 hybrid attn backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-10: imports
```python
from typing import Optional

import torch

from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
from sglang.srt.layers.attention.nsa.nsa_indexer import BaseIndexerMetadata
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode
from sglang.srt.model_executor.model_runner import ModelRunner
from sglang.srt.speculative.spec_info import SpecInput
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 13-15: class HybridAttnBackend
```python
class HybridAttnBackend(AttentionBackend):
    """Support different backends for prefill and decode."""
```
**EN:** Concrete attention backend that connects hybrid attn backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 hybrid attn backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 16-25: method HybridAttnBackend.__init__
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        prefill_backend: AttentionBackend,
        decode_backend: AttentionBackend,
    ):
        self.model_runner = model_runner
        self.prefill_backend = prefill_backend
        self.decode_backend = decode_backend
        self.data_type = model_runner.kv_cache_dtype
```
**EN:** Initializes the HybridAttnBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 HybridAttnBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 27-51: method HybridAttnBackend._select_backend
```python
    def _select_backend(self, forward_mode: ForwardMode) -> AttentionBackend:
        """
        Select the appropriate attention backend based on the forward mode.

        Args:
            forward_mode: The current forward mode indicating the operation type

        Returns:
            The selected attention backend (prefill or decode)

        Note:
            - decode_or_idle: Always uses decode backend
            - target_verify or draft_extend: Uses decode backend if speculative_attention_mode is "decode", otherwise prefill backend
            - prefill: Always uses prefill backend
        """
        if forward_mode.is_decode_or_idle():
            return self.decode_backend
        elif forward_mode.is_target_verify() or forward_mode.is_draft_extend():
            return (
                self.decode_backend
                if self.model_runner.server_args.speculative_attention_mode == "decode"
                else self.prefill_backend
            )
        else:
            return self.prefill_backend
```
**EN:** Implements the select backend routine used by this attention module.
**CN:** 实现该注意力模块使用的 select backend 例程。

### Lines 53-55: method HybridAttnBackend.init_forward_metadata
```python
    def init_forward_metadata(self, forward_batch: ForwardBatch):
        backend = self._select_backend(forward_batch.forward_mode)
        backend.init_forward_metadata(forward_batch)
```
**EN:** Runs the forward-path logic for init forward metadata, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 57-65: method HybridAttnBackend.init_cuda_graph_state
```python
    def init_cuda_graph_state(self, max_bs: int, max_num_tokens: int):
        self.decode_backend.init_cuda_graph_state(max_bs, max_num_tokens)
        if (
            self.model_runner.server_args.speculative_algorithm is not None
            and self.model_runner.server_args.speculative_attention_mode == "prefill"
        ):
            # When speculative decoding is enabled, we need to initialize the backend
            # that will be used for target_verify.
            self.prefill_backend.init_cuda_graph_state(max_bs, max_num_tokens)
```
**EN:** Prepares init cuda graph state so later kernels can execute with the right metadata, layout, and cached state.
**CN:** 准备 init cuda graph state，使后续内核能够使用正确的元数据、布局和缓存状态执行。

### Lines 67-86: method HybridAttnBackend.init_forward_metadata_capture_cuda_graph
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
        backend = self._select_backend(forward_mode)
        backend.init_forward_metadata_capture_cuda_graph(
            bs,
            num_tokens,
            req_pool_indices,
            seq_lens,
            encoder_lens,
            forward_mode,
            spec_info,
        )
```
**EN:** Runs the forward-path logic for init forward metadata capture cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata capture cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 88-109: method HybridAttnBackend.init_forward_metadata_replay_cuda_graph
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
        backend = self._select_backend(forward_mode)
        backend.init_forward_metadata_replay_cuda_graph(
            bs,
            req_pool_indices,
            seq_lens,
            seq_lens_sum,
            encoder_lens,
            forward_mode,
            spec_info,
            seq_lens_cpu,
        )
```
**EN:** Runs the forward-path logic for init forward metadata replay cuda graph, transforming tensors and dispatching the required compute steps.
**CN:** 执行 init forward metadata replay cuda graph 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 111-112: method HybridAttnBackend.get_cuda_graph_seq_len_fill_value
```python
    def get_cuda_graph_seq_len_fill_value(self):
        return self.decode_backend.get_cuda_graph_seq_len_fill_value()
```
**EN:** Computes and returns get cuda graph seq len fill value from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get cuda graph seq len fill value。

### Lines 114-140: method HybridAttnBackend.forward
```python
    def forward(
        self,
        q: Optional[torch.Tensor] = None,  # For full attention
        k: Optional[torch.Tensor] = None,  # For full attention
        v: Optional[torch.Tensor] = None,  # For full attention
        layer: Optional[RadixAttention] = None,
        forward_batch: Optional[ForwardBatch] = None,
        save_kv_cache: bool = True,
        *,
        mixed_qkv: Optional[torch.Tensor] = None,  # For linear attention
        a: Optional[torch.Tensor] = None,  # For linear attention
        b: Optional[torch.Tensor] = None,  # For linear attention
        **kwargs,
    ):
        """Forward method that supports both regular attention (q, k, v) and linear attention (mixed_qkv, a, b)."""
        backend = self._select_backend(forward_batch.forward_mode)
        if mixed_qkv is not None:
            return backend.forward(
                layer=layer,
                forward_batch=forward_batch,
                save_kv_cache=save_kv_cache,
                mixed_qkv=mixed_qkv,
                a=a,
                b=b,
                **kwargs,
            )
        return backend.forward(q, k, v, layer, forward_batch, save_kv_cache, **kwargs)
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 142-154: method HybridAttnBackend.forward_decode
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        **kwargs,
    ):
        return self.decode_backend.forward_decode(
            q, k, v, layer, forward_batch, save_kv_cache, **kwargs
        )
```
**EN:** Runs the forward-path logic for forward decode, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward decode 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 156-169: method HybridAttnBackend.forward_extend
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        **kwargs,
    ):
        backend = self._select_backend(forward_batch.forward_mode)
        return backend.forward_extend(
            q, k, v, layer, forward_batch, save_kv_cache, **kwargs
        )
```
**EN:** Runs the forward-path logic for forward extend, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward extend 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 171-175: method HybridAttnBackend.get_indexer_metadata
```python
    def get_indexer_metadata(
        self, layer_id: int, forward_batch: ForwardBatch
    ) -> Optional[BaseIndexerMetadata]:
        backend = self._select_backend(forward_batch.forward_mode)
        return backend.get_indexer_metadata(layer_id, forward_batch)
```
**EN:** Computes and returns get indexer metadata from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get indexer metadata。

### Lines 177-197: method HybridAttnBackend.forward
```python
    def forward(
        self,
        q: torch.Tensor = None,
        k: torch.Tensor = None,
        v: torch.Tensor = None,
        layer: RadixAttention = None,
        forward_batch: ForwardBatch = None,
        save_kv_cache: bool = True,
        **kwargs,
    ):
        """Delegate forward to the appropriate backend based on forward mode."""
        backend = self._select_backend(forward_batch.forward_mode)
        return backend.forward(
            q=q,
            k=k,
            v=v,
            layer=layer,
            forward_batch=forward_batch,
            save_kv_cache=save_kv_cache,
            **kwargs,
        )
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Compressed or sparse attention helpers / **CN:** 压缩或稀疏注意力辅助逻辑

## Dependencies / 依赖关系
- `typing.Optional`
- `torch`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.layers.attention.nsa.nsa_indexer.BaseIndexerMetadata`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.ForwardMode`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `sglang.srt.speculative.spec_info.SpecInput`
