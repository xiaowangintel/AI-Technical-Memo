# whisper_causal.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/whisper_causal.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Whisper Causal multimodal model adapter used for inference in vLLM. / 实现 Whisper Causal 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-34)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import copy
import functools
import logging
import math
from dataclasses import replace
from functools import partial

import torch
import torch.nn.functional as F
from torch import nn

from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.models.mistral import MistralMLP
from vllm.model_executor.models.whisper import WhisperPosEmbedType
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionMetadata,
    AttentionType,
    CommonAttentionMetadata,
    subclass_attention_backend_with_overrides,
)
from vllm.v1.attention.backends.flash_attn import FlashAttentionBackend
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn.functional supply framework primitives, while internal modules like vllm.config, vllm.distributed, vllm.model_executor.layers.attention, vllm.model_executor.layers.layernorm connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn.functional 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.distributed, vllm.model_executor.layers.attention, vllm.model_executor.layers.layernorm 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Import fallback (lines 35-38)
```python
try:
    from vllm.v1.attention.backends.rocm_aiter_fa import AiterFlashAttentionBackend
except ImportError:
    AiterFlashAttentionBackend = None
```
**EN:** This block provides fallback or compatibility logic so the module can run across multiple environments and dependency versions.
**CN:** 该代码块提供回退或兼容逻辑，使模块可以在多种环境和依赖版本下运行。

### Top-level mapping `logger, CausalRMSNorm` (lines 46-48)
```python
logger = logging.getLogger(__name__)

CausalRMSNorm = partial(RMSNorm, eps=1e-5)
```
**EN:** This assignment block centers on `logger, CausalRMSNorm` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `logger, CausalRMSNorm` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Function `_pad1d` (lines 51-75)
```python
def _pad1d(
    x: torch.Tensor,
    paddings: tuple[int, int],
    mode: str = "constant",
    value: float = 0.0,
) -> torch.Tensor:
    """Tiny wrapper around F.pad, just to allow for
    reflect padding on small input.
    If this is the case, we insert extra 0 padding
    to the right before the reflection happen.
    """
    length = x.shape[-1]
    padding_left, padding_right = paddings
    assert padding_left >= 0 and padding_right >= 0, (padding_left, padding_right)
    if mode == "reflect":
        max_pad = max(padding_left, padding_right)
        extra_pad = 0
        if length <= max_pad:
            extra_pad = max_pad - length + 1
            x = F.pad(x, (0, extra_pad))
        padded = F.pad(x, paddings, mode, value)
        end = padded.shape[-1] - extra_pad
        return padded[..., :end]
    else:
        return F.pad(x, paddings, mode, value)
```
**EN:** The function `_pad1d` helps provide a reusable helper for the surrounding model code. Its main inputs are `x`, `paddings`, `mode`, `value`. Docstring hint: "Tiny wrapper around F.pad, just to allow for reflect padding on small input."
**CN:** 函数 `_pad1d` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `x`、`paddings`、`mode`、`value`。 文档提示：“Tiny wrapper around F.pad, just to allow for reflect padding on small input。”

### Class `WhisperCausalConv1d` (lines 78-109)
```python
class WhisperCausalConv1d(nn.Conv1d):
    def __init__(
        self,
        in_channels: int,
        out_channels: int,
        kernel_size: int,
        stride: int = 1,
        padding: int = 0,
        bias: bool = True,
    ) -> None:
        super().__init__(
            in_channels,
            out_channels,
            kernel_size,
            stride=stride,
            padding=padding,
            bias=bias,
        )
        self._stride = self.stride[0]
        self._effective_kernel_size = (kernel_size - 1) * self.dilation[0] + 1
        self._padding_total = self._effective_kernel_size - self._stride

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        n_frames = (
            x.shape[-1] - self._effective_kernel_size + self._padding_total
        ) / self._stride + 1
        target_length = (math.ceil(n_frames) - 1) * self._stride + (
            self._effective_kernel_size - self._padding_total
        )
        extra_padding = target_length - x.shape[-1]
        x = _pad1d(x, (self._padding_total, extra_padding), mode="constant")
        return super().forward(x)
```
**EN:** Defines `WhisperCausalConv1d`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from nn.Conv1d. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperCausalConv1d`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 nn.Conv1d。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `create_whisper_attention_backend_with_block_pooling` (lines 112-266)
```python
@functools.lru_cache
def create_whisper_attention_backend_with_block_pooling(
    underlying_attn_backend: AttentionBackend, block_pool_size: int
) -> type[AttentionBackend]:
    prefix = "WhisperCausalAttentionWithBlockPooling_"
    underlying_builder = underlying_attn_backend.get_builder_cls()
    underlying_impl = underlying_attn_backend.get_impl_cls()

    class WhisperCausalAttentionWithBlockPoolingBuilder(underlying_builder):  # type: ignore
        def __init__(
            self,
            kv_cache_spec: AttentionSpec,
            layer_names: list[str],
            vllm_config: VllmConfig,
            device: torch.device,
        ):
            assert kv_cache_spec.num_kv_heads % block_pool_size == 0
            kv_cache_spec = replace(
                kv_cache_spec,
                block_size=kv_cache_spec.block_size * block_pool_size,
                num_kv_heads=kv_cache_spec.num_kv_heads // block_pool_size,
            )
            super().__init__(kv_cache_spec, layer_names, vllm_config, device)
            # Override model_config-derived values with the actual
            # encoder values from kv_cache_spec
# ... omitted for brevity ...

    attn_backend = subclass_attention_backend_with_overrides(
        name_prefix=prefix,
        attention_backend_cls=underlying_attn_backend,
        overrides={
            "get_builder_cls": lambda: WhisperCausalAttentionWithBlockPoolingBuilder,
            "get_impl_cls": lambda: WhisperCausalAttentionWithBlockPoolingImpl,
            "get_kv_cache_shape": lambda num_blocks,
            block_size,
            num_kv_heads,
            head_size,
            cache_dtype_str: underlying_attn_backend.get_kv_cache_shape(
                num_blocks,
                # we stretch each block by `block_pool_size`
                block_size * block_pool_size,
                num_kv_heads // block_pool_size,
                head_size,
                cache_dtype_str,
            ),
            "forward_includes_kv_cache_update": True,
        },
    )

    return attn_backend
```
**EN:** The function `create_whisper_attention_backend_with_block_pooling` helps derive pooled or condensed sequence representations. Its main inputs are `underlying_attn_backend`, `block_pool_size`.
**CN:** 函数 `create_whisper_attention_backend_with_block_pooling` 用于生成池化后的序列表示。 它的主要输入包括 `underlying_attn_backend`、`block_pool_size`。

### Class `WhisperCausalAttentionWithBlockPooling` (lines 269-332)
```python
class WhisperCausalAttentionWithBlockPooling(Attention):
    """Attention layer with block pooling."""

    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int | None = None,
        alibi_slopes: list[float] | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        logits_soft_cap: float | None = None,
        per_layer_sliding_window: int | None = None,
        prefix: str = "",
        attn_type: str = AttentionType.DECODER,
        kv_sharing_target_layer_name: str | None = None,
        block_pool_size: int = 1,
        attn_backend: type[AttentionBackend] | None = None,
        **extra_impl_args,
    ) -> None:
        self.block_pool_size = block_pool_size
        dtype = torch.get_default_dtype()

        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
        else:
            kv_cache_dtype = "auto"

        underlying_attn_backend = get_attn_backend(
# ... omitted for brevity ...
        attn_backend = create_whisper_attention_backend_with_block_pooling(
            underlying_attn_backend, block_pool_size
        )

        super().__init__(
            num_heads=num_heads,
            head_size=head_size,
            scale=scale,
            num_kv_heads=num_kv_heads,
            alibi_slopes=alibi_slopes,
            cache_config=cache_config,
            quant_config=quant_config,
            logits_soft_cap=logits_soft_cap,
            per_layer_sliding_window=per_layer_sliding_window,
            prefix=prefix,
            attn_type=attn_type,
            kv_sharing_target_layer_name=kv_sharing_target_layer_name,
            attn_backend=attn_backend,
            **extra_impl_args,
        )

    def get_kv_cache_spec(self, vllm_config: VllmConfig):
        kv_cache_spec = super().get_kv_cache_spec(vllm_config)
        assert isinstance(kv_cache_spec, AttentionSpec)
        kv_cache_spec = replace(
            kv_cache_spec,
            num_kv_heads=self.block_pool_size * kv_cache_spec.num_kv_heads,
        )
        return kv_cache_spec
```
**EN:** Defines `WhisperCausalAttentionWithBlockPooling`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from Attention. Key methods such as `__init__`, `get_kv_cache_spec` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Attention layer with block pooling."
**CN:** 定义 `WhisperCausalAttentionWithBlockPooling`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 Attention。 `__init__`, `get_kv_cache_spec` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Attention layer with block pooling。”

### Class `WhisperCausalAttention` (lines 335-441)
```python
class WhisperCausalAttention(nn.Module):
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        head_dim: int,
        max_position_embeddings: int,
        bias: bool = True,
        attn_type: AttentionType = AttentionType.DECODER,
        per_layer_sliding_window: int | None = None,
        block_pool_size: int = 1,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = embed_dim
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
# ... omitted for brevity ...
    def _init_rotary_emb(self, max_position_embeddings: int) -> None:
        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=max_position_embeddings,
            is_neox_style=False,
            rope_parameters={"rope_theta": 1e6},
        )
# ... omitted for brevity ...
    def _init_qkv(
        self,
        embed_dim: int,
        bias: bool = True,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        self.qkv_proj = QKVParallelLinear(
            hidden_size=embed_dim,
            head_size=self.head_dim,
            total_num_heads=self.total_num_heads,
            total_num_kv_heads=self.total_num_heads,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
        positions: torch.Tensor | None = None,
    ):
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        assert positions is not None
        q, k = self.rotary_emb(positions, q, k)

        attn_output = self.attn(q, k, v)

        output, _ = self.out_proj(attn_output)

        return output
```
**EN:** Defines `WhisperCausalAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `_init_rotary_emb`, `_init_qkv`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperCausalAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `_init_rotary_emb`, `_init_qkv`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `WhisperCausalEncoderLayer` (lines 444-495)
```python
class WhisperCausalEncoderLayer(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        sliding_window = getattr(config, "sliding_window", None)
        block_pool_size = config.block_pool_size
        assert block_pool_size > 1

        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.embed_dim = config.d_model
        self.head_dim = self.embed_dim // config.encoder_attention_heads
        self.self_attn = WhisperCausalAttention(
            embed_dim=self.embed_dim,
            num_heads=config.encoder_attention_heads,
            head_dim=config.encoder_head_dim,
            max_position_embeddings=config.max_position_embeddings,
            block_pool_size=block_pool_size,
            per_layer_sliding_window=sliding_window,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.self_attn_layer_norm = CausalRMSNorm(self.embed_dim)

        self.mlp = MistralMLP(
            hidden_size=config.d_model,
            intermediate_size=config.encoder_ffn_dim,
            hidden_act="silu",
            quant_config=quant_config,
            bias=True,
            gate_up_proj_bias=False,
            prefix=f"{prefix}.mlp",
        )
        self.final_layer_norm = CausalRMSNorm(self.embed_dim)

    def forward(
        self,
        hidden_states: torch.Tensor,
        positions: torch.Tensor | None = None,
    ):
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(hidden_states=hidden_states, positions=positions)
        hidden_states = residual + hidden_states
        residual = hidden_states
        hidden_states = self.final_layer_norm(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** Defines `WhisperCausalEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperCausalEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `WhisperCausalEncoder` (lines 498-547)
```python
class WhisperCausalEncoder(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        embed_dim = config.d_model

        assert WhisperPosEmbedType(config.pos_embed) == WhisperPosEmbedType.ROPE
        assert config.is_causal

        self.num_mel_bins = config.num_mel_bins
        self.max_source_positions = config.max_source_positions
        self.embed_scale = math.sqrt(embed_dim) if config.scale_embedding else 1.0

        self.conv1 = WhisperCausalConv1d(self.num_mel_bins, embed_dim, kernel_size=3)
        self.conv2 = WhisperCausalConv1d(embed_dim, embed_dim, stride=2, kernel_size=3)

        self.total_stride = self.conv1.stride[0] * self.conv2.stride[0]
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.encoder_layers,
            lambda prefix: WhisperCausalEncoderLayer(
                vllm_config=vllm_config, prefix=f"{prefix}.layers"
            ),
            prefix=f"{prefix}.layers",
        )
        self.layer_norm = CausalRMSNorm(config.d_model)

    def forward_conv(
        self, input_features: torch.Tensor | list[torch.Tensor]
    ) -> torch.Tensor:
        hidden_states = []
        for features in input_features:
            embeds = nn.functional.gelu(self.conv1(features))
            embeds = nn.functional.gelu(self.conv2(embeds))

            embeds = embeds.transpose(-1, -2).to(embeds.dtype)

            hidden_states.append(embeds)

        hidden_states = torch.cat(hidden_states)

        return hidden_states

    def forward(
        self, hidden_states: torch.Tensor, positions: torch.Tensor
    ) -> torch.Tensor:
        for encoder_layer in self.layers:
            hidden_states = encoder_layer(hidden_states, positions)

        hidden_states = self.layer_norm(hidden_states)
        return hidden_states
```
**EN:** Defines `WhisperCausalEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward_conv`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `WhisperCausalEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward_conv`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Distributed execution**
  - **EN:** The implementation is aware of tensor parallelism or pipeline parallelism, so layers and weights can be sharded across devices.
  - **CN:** 实现考虑了张量并行或流水线并行，因此层和权重可以在多设备间切分。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Pooling and encoding**
  - **EN:** This file exposes encoder-style behavior, pooled outputs, or sequence-level representations beyond causal decoding.
  - **CN:** 该文件提供编码器式行为、池化输出，或超出因果解码的序列表征。

## Dependencies / 依赖关系
- **Standard library**: copy, functools, logging, math, dataclasses
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn.functional
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.distributed, vllm.model_executor.layers.attention, vllm.model_executor.layers.layernorm, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization, vllm.model_executor.layers.rotary_embedding, vllm.model_executor.models.mistral
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
