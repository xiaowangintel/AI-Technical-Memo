# mimo_audio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/mimo_audio.py`
- **Repository**: sgl-project/sglang
- **Purpose**: MiMo audio: tokenizer, encoding utilities, and audio encoder. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module docstring
```python
"""MiMo audio: tokenizer, encoding utilities, and audio encoder."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 5-25: Module imports
```python
import logging
import math
import os
import typing as tp
from dataclasses import dataclass
from functools import wraps
from typing import List, Optional, Tuple

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange
from transformers.activations import ACT2FN
from transformers.configuration_utils import PretrainedConfig
from transformers.modeling_utils import PreTrainedModel
from transformers.models.qwen2.configuration_qwen2 import Qwen2Config
from transformers.models.qwen2.modeling_qwen2 import Qwen2Model

from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import is_cuda
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 27-32: Top-level if
```python
if is_cuda():
    from sgl_kernel.flash_attn import flash_attn_varlen_func
else:

    def flash_attn_varlen_func(*args, **kwargs):
        raise RuntimeError("MiMoAudioTokenizer requires CUDA to run.")
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 35-35: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 38-73: Function `_compute_default_rope_parameters`
```python
def _compute_default_rope_parameters(
    config=None, device=None, seq_len=None, **rope_kwargs
):
    if config is not None and len(rope_kwargs) > 0:
        raise ValueError(
            "Unexpected arguments: `**rope_kwargs` and `config` are mutually exclusive"
        )
    if len(rope_kwargs) > 0:
        base = rope_kwargs["base"]
        dim = rope_kwargs["dim"]
    elif config is not None:
        base = config.rope_theta
        partial_rotary_factor = (
            config.partial_rotary_factor
            if hasattr(config, "partial_rotary_factor")
            else 1.0
        )
        head_dim = getattr(config, "head_dim", None)
        if head_dim is None:
            head_dim = config.hidden_size // config.num_attention_heads
            logger.info(
                "audio.head_dim not set; defaulting to hidden_size/num_heads = %d",
                head_dim,
            )
        dim = int(head_dim * partial_rotary_factor)
    attention_factor = 1.0
    inv_freq = 1.0 / (
        base
        ** (
            torch.arange(0, dim, 2, dtype=torch.int64).to(
                device=device, dtype=torch.float
            )
            / dim
        )
    )
    return inv_freq, attention_factor
```
**EN:** This function implements `_compute_default_rope_parameters(config=..., device=..., seq_len=..., **rope_kwargs)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_compute_default_rope_parameters(config=..., device=..., seq_len=..., **rope_kwargs)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 76-78: Top-level assign
```python
_ROPE_INIT_FUNCTIONS = {
    "default": _compute_default_rope_parameters,
}
```
**EN:** Defines or updates _ROPE_INIT_FUNCTIONS, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _ROPE_INIT_FUNCTIONS，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 81-125: Function `_dynamic_rope_update`
```python
def _dynamic_rope_update(rope_forward):
    def longrope_frequency_update(self, position_ids, device):
        seq_len = torch.max(position_ids) + 1
        if hasattr(self.config, "original_max_position_embeddings"):
            original_max_position_embeddings = (
                self.config.original_max_position_embeddings
            )
        else:
            original_max_position_embeddings = self.config.max_position_embeddings
        if seq_len > original_max_position_embeddings:
            if not hasattr(self, "long_inv_freq"):
                self.long_inv_freq, _ = self.rope_init_fn(
                    self.config, device, seq_len=original_max_position_embeddings + 1
                )
            self.register_buffer("inv_freq", self.long_inv_freq, persistent=False)
        else:
            self.original_inv_freq = self.original_inv_freq.to(device)
            self.register_buffer("inv_freq", self.original_inv_freq, persistent=False)

    def dynamic_frequency_update(self, position_ids, device):
        seq_len = torch.max(position_ids) + 1
        if seq_len > self.max_seq_len_cached:  # growth
            inv_freq, self.attention_scaling = self.rope_init_fn(
                self.config, device, seq_len=seq_len
            )
            self.register_buffer("inv_freq", inv_freq, persistent=False)
            self.max_seq_len_cached = seq_len

        if (
            seq_len < self.original_max_seq_len
            and self.max_seq_len_cached > self.original_max_seq_len
        ):
            self.original_inv_freq = self.original_inv_freq.to(device)
            self.register_buffer("inv_freq", self.original_inv_freq, persistent=False)
            self.max_seq_len_cached = self.original_max_seq_len

    @wraps(rope_forward)
    def wrapper(self, x, position_ids):
        if "dynamic" in self.rope_type:
            dynamic_frequency_update(self, position_ids, device=x.device)
# ... truncated for brevity ...
```
**EN:** This function implements `_dynamic_rope_update(rope_forward)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_dynamic_rope_update(rope_forward)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 128-128: Class `AudioRotaryEmbedding` overview
```python
class AudioRotaryEmbedding(nn.Module):
```
**EN:** Defines `AudioRotaryEmbedding` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioRotaryEmbedding`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 129-138: Method `AudioRotaryEmbedding.__init__`
```python
    def __init__(self, base, dim, max_seq_len, rope_type="default", device=None):
        super().__init__()
        self.max_seq_len = max_seq_len
        self.rope_type = rope_type
        self.rope_init_fn = _ROPE_INIT_FUNCTIONS[self.rope_type]
        inv_freq, self.attention_scaling = self.rope_init_fn(
            device=device, base=base, dim=dim
        )
        self.register_buffer("inv_freq", inv_freq, persistent=False)
        self.original_inv_freq = self.inv_freq
```
**EN:** This method implements `__init__(base, dim, max_seq_len, rope_type=..., device=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(base, dim, max_seq_len, rope_type=..., device=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 140-157: Method `AudioRotaryEmbedding.forward`
```python
    @torch.no_grad()
    @_dynamic_rope_update
    def forward(self, x, position_ids):
        inv_freq_expanded = self.inv_freq[:, None].float().expand(-1, 1).to(x.device)
        position_ids_expanded = position_ids[None, :].float()
        device_type = (
            x.device.type
            if isinstance(x.device.type, str) and x.device.type != "mps"
            else "cpu"
        )
        with torch.autocast(device_type=device_type, enabled=False):  # Force float32
            freqs = (
                inv_freq_expanded.float() @ position_ids_expanded.float()
            ).transpose(0, 1)
            emb = torch.cat((freqs, freqs), dim=-1)
            cos = emb.cos() * self.attention_scaling
            sin = emb.sin() * self.attention_scaling
        return cos.to(dtype=x.dtype), sin.to(dtype=x.dtype)
```
**EN:** This method implements `forward(x, position_ids)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad(), _dynamic_rope_update.
**CN:** 这个方法实现了 `forward(x, position_ids)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad(), _dynamic_rope_update。

### Lines 160-162: Class `EuclideanCodebook` overview
```python
class EuclideanCodebook(nn.Module):
    """Codebook with Euclidean distance (inference-only)."""
```
**EN:** Defines `EuclideanCodebook` as a reusable runtime type derived from nn.Module. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `EuclideanCodebook`，其继承关系为 nn.Module。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 163-175: Method `EuclideanCodebook.__init__`
```python
    def __init__(
        self, dim: int, codebook_size: int, kmeans_init: bool = False, **kwargs
    ):
        super().__init__()
        init_fn = self._uniform_init if not kmeans_init else torch.zeros
        embed = init_fn(codebook_size, dim)

        self.codebook_size = codebook_size

        self.register_buffer("inited", torch.Tensor([not kmeans_init]))
        self.register_buffer("cluster_size", torch.zeros(codebook_size))
        self.register_buffer("embed", embed)
        self.register_buffer("embed_avg", embed.clone())
```
**EN:** This method implements `__init__(dim: ..., codebook_size: ..., kmeans_init: ...=..., **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., codebook_size: ..., kmeans_init: ...=..., **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 177-179: Method `EuclideanCodebook.preprocess`
```python
    def preprocess(self, x):
        x = rearrange(x, "... d -> (...) d")
        return x
```
**EN:** This method implements `preprocess(x)` and implements a focused helper that supports the surrounding runtime flow inside `EuclideanCodebook`.
**CN:** 这个方法实现了 `preprocess(x)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `EuclideanCodebook` 内部调用。

### Lines 181-189: Method `EuclideanCodebook.quantize`
```python
    def quantize(self, x):
        embed = self.embed.t()
        dist_val = -(
            x.pow(2).sum(1, keepdim=True)
            - 2 * x @ embed
            + embed.pow(2).sum(0, keepdim=True)
        )
        embed_ind = dist_val.max(dim=-1).indices
        return embed_ind
```
**EN:** This method implements `quantize(x)` and implements a focused helper that supports the surrounding runtime flow inside `EuclideanCodebook`.
**CN:** 这个方法实现了 `quantize(x)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `EuclideanCodebook` 内部调用。

### Lines 191-192: Method `EuclideanCodebook.postprocess_emb`
```python
    def postprocess_emb(self, embed_ind, shape):
        return embed_ind.view(*shape[:-1])
```
**EN:** This method implements `postprocess_emb(embed_ind, shape)` and implements a focused helper that supports the surrounding runtime flow inside `EuclideanCodebook`.
**CN:** 这个方法实现了 `postprocess_emb(embed_ind, shape)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `EuclideanCodebook` 内部调用。

### Lines 194-196: Method `EuclideanCodebook.dequantize`
```python
    def dequantize(self, embed_ind):
        quantize = F.embedding(embed_ind, self.embed)
        return quantize
```
**EN:** This method implements `dequantize(embed_ind)` and implements a focused helper that supports the surrounding runtime flow inside `EuclideanCodebook`.
**CN:** 这个方法实现了 `dequantize(embed_ind)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `EuclideanCodebook` 内部调用。

### Lines 198-203: Method `EuclideanCodebook.encode`
```python
    def encode(self, x):
        shape = x.shape
        x = self.preprocess(x)
        embed_ind = self.quantize(x)
        embed_ind = self.postprocess_emb(embed_ind, shape)
        return embed_ind
```
**EN:** This method implements `encode(x)` and implements a focused helper that supports the surrounding runtime flow inside `EuclideanCodebook`.
**CN:** 这个方法实现了 `encode(x)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `EuclideanCodebook` 内部调用。

### Lines 205-207: Method `EuclideanCodebook.decode`
```python
    def decode(self, embed_ind):
        quantize = self.dequantize(embed_ind)
        return quantize
```
**EN:** This method implements `decode(embed_ind)` and implements a focused helper that supports the surrounding runtime flow inside `EuclideanCodebook`.
**CN:** 这个方法实现了 `decode(embed_ind)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `EuclideanCodebook` 内部调用。

### Lines 209-213: Method `EuclideanCodebook._uniform_init`
```python
    @staticmethod
    def _uniform_init(*shape: int):
        t = torch.empty(shape)
        nn.init.kaiming_uniform_(t)
        return t
```
**EN:** This method implements `_uniform_init(*shape)` and performs setup work for later runtime execution Decorators: staticmethod.
**CN:** 这个方法实现了 `_uniform_init(*shape)`，其作用是执行后续运行时所需的初始化工作 装饰器：staticmethod。

### Lines 216-218: Class `VectorQuantization` overview
```python
class VectorQuantization(nn.Module):
    """Vector quantization with euclidean distance (inference-only)."""
```
**EN:** Defines `VectorQuantization` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `VectorQuantization`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 219-243: Method `VectorQuantization.__init__`
```python
    def __init__(
        self,
        dim: int,
        codebook_size: int,
        codebook_dim: tp.Optional[int] = None,
        kmeans_init: bool = True,
        **kwargs,
    ):
        super().__init__()
        _codebook_dim: int = codebook_dim if codebook_dim is not None else dim

        requires_projection = _codebook_dim != dim
        self.project_in = (
            nn.Linear(dim, _codebook_dim) if requires_projection else nn.Identity()
        )
        self.project_out = (
            nn.Linear(_codebook_dim, dim) if requires_projection else nn.Identity()
        )

        self._codebook = EuclideanCodebook(
            dim=_codebook_dim,
            codebook_size=codebook_size,
            kmeans_init=kmeans_init,
        )
        self.codebook_size = codebook_size
```
**EN:** This method implements `__init__(dim: ..., codebook_size: ..., codebook_dim: ...=..., kmeans_init: ...=..., **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., codebook_size: ..., codebook_dim: ...=..., kmeans_init: ...=..., **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 245-247: Method `VectorQuantization.codebook`
```python
    @property
    def codebook(self):
        return self._codebook.embed
```
**EN:** This method implements `codebook()` and implements a focused helper that supports the surrounding runtime flow inside `VectorQuantization` Decorators: property.
**CN:** 这个方法实现了 `codebook()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `VectorQuantization` 内部调用 装饰器：property。

### Lines 249-252: Method `VectorQuantization.encode`
```python
    def encode(self, x):
        x = self.project_in(x)
        embed_in = self._codebook.encode(x)
        return embed_in
```
**EN:** This method implements `encode(x)` and implements a focused helper that supports the surrounding runtime flow inside `VectorQuantization`.
**CN:** 这个方法实现了 `encode(x)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `VectorQuantization` 内部调用。

### Lines 254-257: Method `VectorQuantization.decode`
```python
    def decode(self, embed_ind):
        quantize = self._codebook.decode(embed_ind)
        quantize = self.project_out(quantize)
        return quantize
```
**EN:** This method implements `decode(embed_ind)` and implements a focused helper that supports the surrounding runtime flow inside `VectorQuantization`.
**CN:** 这个方法实现了 `decode(embed_ind)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `VectorQuantization` 内部调用。

### Lines 260-264: Class `ResidualVectorQuantization` overview
```python
class ResidualVectorQuantization(nn.Module):
    """Residual vector quantization implementation.
    Follows Algorithm 1. in https://arxiv.org/pdf/2107.03312.pdf
    """
```
**EN:** Defines `ResidualVectorQuantization` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ResidualVectorQuantization`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 265-276: Method `ResidualVectorQuantization.__init__`
```python
    def __init__(self, *, num_quantizers, codebook_size, **kwargs):
        super().__init__()
        if isinstance(codebook_size, int):
            codebook_size = [codebook_size] * num_quantizers
        elif len(codebook_size) < num_quantizers:
            codebook_size += [codebook_size[-1]] * (num_quantizers - len(codebook_size))
        self.layers = nn.ModuleList(
            [
                VectorQuantization(codebook_size=codebook_size[i], **kwargs)
                for i in range(num_quantizers)
            ]
        )
```
**EN:** This method implements `__init__(*, num_quantizers, codebook_size, **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, num_quantizers, codebook_size, **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 278-291: Method `ResidualVectorQuantization.encode`
```python
    def encode(
        self, x: torch.Tensor, n_q: tp.Optional[int] = None, st: tp.Optional[int] = None
    ) -> torch.Tensor:
        residual = x
        all_indices = []
        n_q = len(self.layers) if n_q is None else n_q
        st = 0 if st is None else st
        for layer in self.layers[st:n_q]:
            indices = layer.encode(residual)
            quantized = layer.decode(indices)
            residual = residual - quantized
            all_indices.append(indices)
        out_indices = torch.stack(all_indices)
        return out_indices
```
**EN:** This method implements `encode(x: ..., n_q: ...=..., st: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `ResidualVectorQuantization`.
**CN:** 这个方法实现了 `encode(x: ..., n_q: ...=..., st: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ResidualVectorQuantization` 内部调用。

### Lines 293-299: Method `ResidualVectorQuantization.decode`
```python
    def decode(self, q_indices: torch.Tensor, st: int = 0) -> torch.Tensor:
        quantized_out = self.layers[st].decode(q_indices[0])
        for i in range(1, len(q_indices)):
            layer = self.layers[st + i]
            quantized = layer.decode(q_indices[i])
            quantized_out = quantized_out + quantized
        return quantized_out
```
**EN:** This method implements `decode(q_indices: ..., st: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `ResidualVectorQuantization`.
**CN:** 这个方法实现了 `decode(q_indices: ..., st: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ResidualVectorQuantization` 内部调用。

### Lines 302-304: Class `ResidualVectorQuantizer` overview
```python
class ResidualVectorQuantizer(nn.Module):
    """Residual Vector Quantizer (inference-only)."""
```
**EN:** Defines `ResidualVectorQuantizer` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ResidualVectorQuantizer`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 305-320: Method `ResidualVectorQuantizer.__init__`
```python
    def __init__(
        self,
        dimension: int = 256,
        n_q: int = 8,
        bins: int | list = 1024,
        kmeans_init: bool = True,
        **kwargs,
    ):
        super().__init__()
        self.n_q = n_q
        self.vq = ResidualVectorQuantization(
            dim=dimension,
            codebook_size=bins,
            num_quantizers=n_q,
            kmeans_init=kmeans_init,
        )
```
**EN:** This method implements `__init__(dimension: ...=..., n_q: ...=..., bins: ...=..., kmeans_init: ...=..., **kwargs)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dimension: ...=..., n_q: ...=..., bins: ...=..., kmeans_init: ...=..., **kwargs)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 322-328: Method `ResidualVectorQuantizer.encode`
```python
    def encode(
        self, x: torch.Tensor, n_q: tp.Optional[int] = None, st: tp.Optional[int] = None
    ) -> torch.Tensor:
        n_q = n_q if n_q else self.n_q
        st = st or 0
        codes = self.vq.encode(x, n_q=n_q, st=st)
        return codes
```
**EN:** This method implements `encode(x: ..., n_q: ...=..., st: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `ResidualVectorQuantizer`.
**CN:** 这个方法实现了 `encode(x: ..., n_q: ...=..., st: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ResidualVectorQuantizer` 内部调用。

### Lines 330-332: Method `ResidualVectorQuantizer.decode`
```python
    def decode(self, codes: torch.Tensor, st: int = 0) -> torch.Tensor:
        quantized = self.vq.decode(codes, st=st)
        return quantized
```
**EN:** This method implements `decode(codes: ..., st: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `ResidualVectorQuantizer`.
**CN:** 这个方法实现了 `decode(codes: ..., st: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `ResidualVectorQuantizer` 内部调用。

### Lines 335-335: Class `MiMoAudioTokenizerConfig` overview
```python
class MiMoAudioTokenizerConfig(PretrainedConfig):
```
**EN:** Defines `MiMoAudioTokenizerConfig` as a reusable runtime type derived from PretrainedConfig. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoAudioTokenizerConfig`，其继承关系为 PretrainedConfig。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 336-336: Class `MiMoAudioTokenizerConfig` attributes
```python
    model_type = "mimo_audio_tokenizer"
```
**EN:** Defines class-level attributes and metadata that shape how `MiMoAudioTokenizerConfig` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiMoAudioTokenizerConfig` 在运行时的行为。

### Lines 338-442: Method `MiMoAudioTokenizerConfig.__init__`
```python
    def __init__(
        self,
        max_audio_seconds: int = 1800,
        stride_size: int = 2,
        avg_pooler: int = 1,
        d_model: int = 768,
        scale_embedding: bool = True,
        kernel_size: int = 3,
        activation_function: str = "gelu",
        encoder_layers: int = 8,
        encoder_skip_layer_id: int = None,
        encoder_attention_heads: int = 12,
        encoder_ffn_dim: int = 3072,
        encoder_causal: bool = False,
        encoder_attn_window_size: list = None,
        decoder_layers: int = 8,
        decoder_attention_heads: int = 12,
        decoder_ffn_dim: int = 3072,
        decoder_kernel_size: int = 3,
        decoder_stride_size: int = 2,
        decoder_causal: bool = True,
        decoder_attn_window_size: list = None,
        nfft: int = 1024,
        vocoder_dim: int = 512,
        vocoder_intermediate_dim: int = 4096,
        vocoder_num_layers: int = 30,
        n_mels: int = 80,
        sampling_rate: int = 24000,
        hop_length: int = 240,
        window_size: int = 1024,
        vocoder_padding: str = "same",
        fmin: int = 0,
        fmax: int = None,
        num_quantizers: int = 12,
        codebook_size: list = None,
        threshold_ema_dead_code: int = 10,
        position_embedding_type: str = "rope",
        rope_theta: int = 10000,
        rope_type: str = "default",
        ln_type: str = "LayerNorm",
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(max_audio_seconds: ...=..., stride_size: ...=..., avg_pooler: ...=..., d_model: ...=..., scale_embedding: ...=..., kernel_size: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(max_audio_seconds: ...=..., stride_size: ...=..., avg_pooler: ...=..., d_model: ...=..., scale_embedding: ...=..., kernel_size: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 445-455: Function `get_sequence_mask`
```python
def get_sequence_mask(inputs, inputs_length):
    if inputs.dim() == 3:
        bsz, tgt_len, _ = inputs.size()
    else:
        bsz, tgt_len = inputs_length.shape[0], torch.max(inputs_length)
    sequence_mask = torch.arange(0, tgt_len).to(inputs.device)
    sequence_mask = torch.lt(sequence_mask, inputs_length.reshape(bsz, 1)).view(
        bsz, tgt_len, 1
    )
    unpacking_index = torch.cumsum(sequence_mask.to(torch.int64).view(-1), dim=0) - 1
    return sequence_mask, unpacking_index
```
**EN:** This function implements `get_sequence_mask(inputs, inputs_length)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个函数实现了 `get_sequence_mask(inputs, inputs_length)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 458-467: Function `unpack_hidden_states`
```python
def unpack_hidden_states(
    hidden_states, lengths, sequence_mask=None, unpacking_index=None
):
    bsz = lengths.shape[0]
    if sequence_mask is None or unpacking_index is None:
        sequence_mask, unpacking_index = get_sequence_mask(hidden_states, lengths)
    hidden_states = torch.index_select(hidden_states, 0, unpacking_index).view(
        bsz, torch.max(lengths), hidden_states.shape[-1]
    )
    return torch.where(sequence_mask, hidden_states, 0)
```
**EN:** This function implements `unpack_hidden_states(hidden_states, lengths, sequence_mask=..., unpacking_index=...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `unpack_hidden_states(hidden_states, lengths, sequence_mask=..., unpacking_index=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 470-474: Function `get_position_ids`
```python
def get_position_ids(lengths):
    total_len = lengths.sum()
    offset = torch.cat([torch.zeros(1).to(lengths), lengths[:-1].cumsum(dim=0)])
    offset = torch.repeat_interleave(offset, lengths)
    return torch.arange(0, total_len).to(offset) - offset
```
**EN:** This function implements `get_position_ids(lengths)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个函数实现了 `get_position_ids(lengths)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 477-477: Top-level assign
```python
LAYER_NORM = {"LayerNorm": nn.LayerNorm}
```
**EN:** Defines or updates LAYER_NORM, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 LAYER_NORM，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 480-480: Class `AudioEncoderAttention` overview
```python
class AudioEncoderAttention(nn.Module):
```
**EN:** Defines `AudioEncoderAttention` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioEncoderAttention`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 481-498: Method `AudioEncoderAttention.__init__`
```python
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        window_size: Tuple[int, int] = (-1, -1),
        causal: bool = False,
    ):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.window_size = window_size
        self.causal = causal

        self.k_proj = nn.Linear(embed_dim, embed_dim, bias=False)
        self.v_proj = nn.Linear(embed_dim, embed_dim, bias=True)
        self.q_proj = nn.Linear(embed_dim, embed_dim, bias=True)
        self.out_proj = nn.Linear(embed_dim, embed_dim, bias=True)
```
**EN:** This method implements `__init__(embed_dim: ..., num_heads: ..., window_size: ...=..., causal: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(embed_dim: ..., num_heads: ..., window_size: ...=..., causal: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 500-537: Method `AudioEncoderAttention.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        max_seqlen: int,
        rope_position_embeddings=None,
    ):
        bsz, _ = hidden_states.size()

        query_states = self.q_proj(hidden_states).view(
            bsz, self.num_heads, self.head_dim
        )
        key_states = self.k_proj(hidden_states).view(bsz, self.num_heads, self.head_dim)
        value_states = self.v_proj(hidden_states).view(
            bsz, self.num_heads, self.head_dim
        )

        if rope_position_embeddings is not None:
            cos, sin = rope_position_embeddings
            query_states, key_states = self.apply_rotary_pos_emb(
                query_states, key_states, cos, sin
            )

        attn_output = flash_attn_varlen_func(
            query_states,
            key_states,
            value_states,
            cu_seqlens,
            cu_seqlens,
            max_seqlen,
            max_seqlen,
            causal=self.causal,
            window_size=self.window_size,
        )

        attn_output = attn_output.reshape(bsz, self.embed_dim)
        attn_output = self.out_proj(attn_output)
        return attn_output
```
**EN:** This method implements `forward(hidden_states: ..., cu_seqlens: ..., max_seqlen: ..., rope_position_embeddings=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., cu_seqlens: ..., max_seqlen: ..., rope_position_embeddings=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 539-543: Method `AudioEncoderAttention._rotate_half`
```python
    @staticmethod
    def _rotate_half(x):
        x1 = x[..., : x.shape[-1] // 2]
        x2 = x[..., x.shape[-1] // 2 :]
        return torch.cat((-x2, x1), dim=-1)
```
**EN:** This method implements `_rotate_half()` and implements a focused helper that supports the surrounding runtime flow inside `AudioEncoderAttention` Decorators: staticmethod.
**CN:** 这个方法实现了 `_rotate_half()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AudioEncoderAttention` 内部调用 装饰器：staticmethod。

### Lines 545-551: Method `AudioEncoderAttention.apply_rotary_pos_emb`
```python
    @classmethod
    def apply_rotary_pos_emb(cls, q, k, cos, sin, unsqueeze_dim=1):
        cos = cos.unsqueeze(unsqueeze_dim)
        sin = sin.unsqueeze(unsqueeze_dim)
        q_embed = (q * cos) + (cls._rotate_half(q) * sin)
        k_embed = (k * cos) + (cls._rotate_half(k) * sin)
        return q_embed, k_embed
```
**EN:** This method implements `apply_rotary_pos_emb(q, k, cos, sin, unsqueeze_dim=...)` and implements a focused helper that supports the surrounding runtime flow inside `AudioEncoderAttention` Decorators: classmethod.
**CN:** 这个方法实现了 `apply_rotary_pos_emb(q, k, cos, sin, unsqueeze_dim=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AudioEncoderAttention` 内部调用 装饰器：classmethod。

### Lines 554-554: Class `AudioEncoderTransformerLayer` overview
```python
class AudioEncoderTransformerLayer(nn.Module):
```
**EN:** Defines `AudioEncoderTransformerLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioEncoderTransformerLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 555-575: Method `AudioEncoderTransformerLayer.__init__`
```python
    def __init__(
        self,
        config: MiMoAudioTokenizerConfig,
        causal: bool,
        attn_window_size: Tuple[int, int] = (-1, -1),
    ):
        super().__init__()
        self.embed_dim = config.d_model

        self.self_attn = AudioEncoderAttention(
            embed_dim=self.embed_dim,
            num_heads=config.encoder_attention_heads,
            window_size=attn_window_size,
            causal=causal,
        )
        self.self_attn_layer_norm = LAYER_NORM[config.ln_type](self.embed_dim)

        self.activation_fn = ACT2FN[config.activation_function]
        self.fc1 = nn.Linear(self.embed_dim, config.encoder_ffn_dim)
        self.fc2 = nn.Linear(config.encoder_ffn_dim, self.embed_dim)
        self.final_layer_norm = LAYER_NORM[config.ln_type](self.embed_dim)
```
**EN:** This method implements `__init__(config: ..., causal: ..., attn_window_size: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., causal: ..., attn_window_size: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 577-600: Method `AudioEncoderTransformerLayer.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        max_seqlen: int,
        rope_position_embeddings: Tuple[torch.Tensor, torch.Tensor],
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        hidden_states = self.self_attn(
            hidden_states,
            cu_seqlens,
            max_seqlen,
            rope_position_embeddings=rope_position_embeddings,
        )
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.final_layer_norm(hidden_states)
        hidden_states = self.activation_fn(self.fc1(hidden_states))
        hidden_states = self.fc2(hidden_states)
        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ..., cu_seqlens: ..., max_seqlen: ..., rope_position_embeddings: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., cu_seqlens: ..., max_seqlen: ..., rope_position_embeddings: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 603-603: Class `AudioEncoder` overview
```python
class AudioEncoder(nn.Module):
```
**EN:** Defines `AudioEncoder` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioEncoder`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 604-685: Method `AudioEncoder.__init__`
```python
    def __init__(
        self,
        config: MiMoAudioTokenizerConfig,
    ):
        super().__init__()
        self.config = config
        self.max_source_positions = (
            config.max_audio_seconds * config.sampling_rate // config.hop_length
        ) // config.stride_size
        self.embed_scale = math.sqrt(config.d_model) if config.scale_embedding else 1.0
        self.skip_layer_idx = config.encoder_skip_layer_id

        self.conv1 = nn.Conv1d(
            config.n_mels,
            config.d_model,
            kernel_size=config.kernel_size,
            padding=1,
        )
        self.conv2 = nn.Conv1d(
            config.d_model,
            config.d_model,
            kernel_size=config.kernel_size,
            stride=config.stride_size,
            padding=1,
        )

        self.position_embedding = AudioRotaryEmbedding(
            config.rope_theta,
            config.d_model // config.encoder_attention_heads,
            self.max_source_positions,
            config.rope_type,
        )

        attn_window_sizes = []
        if config.hybrid_attention:
            for i in range(config.encoder_layers):
                if i % config.swa_per_block < config.swa_per_block - 1:
                    attn_window_sizes.append(tuple(config.encoder_attn_window_size))
                else:
                    attn_window_sizes.append((-1, -1))
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 687-759: Method `AudioEncoder.get_features`
```python
    def get_features(self, input_features, output_length):
        input_features = input_features.to(self.conv1.weight)
        inputs_embeds = nn.functional.gelu(self.conv1(input_features))
        inputs_embeds = nn.functional.gelu(self.conv2(inputs_embeds))
        inputs_embeds = inputs_embeds.permute(0, 2, 1)
        bsz, tgt_len, _ = inputs_embeds.size()
        hidden_states = inputs_embeds

        position_ids = get_position_ids(output_length).long().to(input_features.device)
        rope_position_embeddings = self.position_embedding(input_features, position_ids)

        attention_mask, unpacking_index = get_sequence_mask(
            hidden_states, output_length
        )
        hidden_states = torch.masked_select(hidden_states, attention_mask).view(
            torch.sum(output_length), self.config.d_model
        )

        cu_seqlens = F.pad(
            torch.cumsum(output_length, dim=0), (1, 0), "constant", 0
        ).to(device=hidden_states.device, dtype=torch.int32)
        max_seqlen = torch.max(output_length).to(torch.int32).item()

        skip_connect_hidden_states = 0.0
        for idx, encoder_layer in enumerate(self.layers):
            hidden_states = encoder_layer(
                hidden_states,
                cu_seqlens,
                max_seqlen,
                rope_position_embeddings=rope_position_embeddings,
            )
            if (self.skip_layer_idx is not None) and idx == self.skip_layer_idx - 1:
                skip_connect_hidden_states = hidden_states.clone()

        hidden_states += skip_connect_hidden_states
        hidden_states = self.layer_norm(hidden_states)

        if self.down_sample_layer is not None:
            hidden_states = torch.index_select(hidden_states, 0, unpacking_index).view(
                bsz, tgt_len, self.config.d_model
# ... truncated for brevity ...
```
**EN:** This method implements `get_features(input_features, output_length)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_features(input_features, output_length)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 761-763: Method `AudioEncoder.get_output_length`
```python
    def get_output_length(self, mel_len):
        tgt_len = mel_len + 3 - self.config.kernel_size
        return (tgt_len + 2 - self.config.kernel_size) // self.config.stride_size + 1
```
**EN:** This method implements `get_output_length(mel_len)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_output_length(mel_len)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 765-801: Method `AudioEncoder.encode`
```python
    @torch.no_grad()
    def encode(
        self,
        input_features,
        input_lens=None,
        output_length=None,
        return_codes_only=False,
        n_q=None,
        use_quantizer=True,
    ):
        if output_length is None:
            output_length = self.get_output_length(input_lens)
        input_features = unpack_hidden_states(input_features, input_lens)
        hidden_states, output_length, attention_mask, unpacking_index, tgt_len, bsz = (
            self.get_features(
                input_features=input_features.transpose(1, 2),
                output_length=output_length,
            )
        )

        dtype = hidden_states.dtype
        if use_quantizer and self.quantizer is not None:
            self.quantizer.float()
            codes = self.quantizer.encode(hidden_states.float(), n_q=n_q)
            if return_codes_only:
                return codes, output_length
            hidden_states = self.quantizer.decode(codes)
            hidden_states = hidden_states.to(dtype)
        else:
            codes = None

        hidden_states_packed = hidden_states.clone()
        hidden_states = torch.index_select(hidden_states, 0, unpacking_index).view(
            bsz, tgt_len, self.config.d_model
        )
        hidden_states = torch.where(attention_mask, hidden_states, 0)
        return hidden_states, hidden_states_packed, output_length, codes
```
**EN:** This method implements `encode(input_features, input_lens=..., output_length=..., return_codes_only=..., n_q=..., use_quantizer=...)` and implements a focused helper that supports the surrounding runtime flow inside `AudioEncoder` Decorators: torch.no_grad().
**CN:** 这个方法实现了 `encode(input_features, input_lens=..., output_length=..., return_codes_only=..., n_q=..., use_quantizer=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AudioEncoder` 内部调用 装饰器：torch.no_grad()。

### Lines 803-806: Method `AudioEncoder.decode_vq`
```python
    @torch.no_grad()
    def decode_vq(self, codes):
        self.quantizer.float()
        return self.quantizer.decode(codes)
```
**EN:** This method implements `decode_vq(codes)` and implements a focused helper that supports the surrounding runtime flow inside `AudioEncoder` Decorators: torch.no_grad().
**CN:** 这个方法实现了 `decode_vq(codes)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AudioEncoder` 内部调用 装饰器：torch.no_grad()。

### Lines 809-809: Class `MiMoAudioTokenizer` overview
```python
class MiMoAudioTokenizer(PreTrainedModel):
```
**EN:** Defines `MiMoAudioTokenizer` as a reusable runtime type derived from PreTrainedModel. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoAudioTokenizer`，其继承关系为 PreTrainedModel。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 810-810: Class `MiMoAudioTokenizer` attributes
```python
    config_class = MiMoAudioTokenizerConfig
```
**EN:** Defines class-level attributes and metadata that shape how `MiMoAudioTokenizer` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiMoAudioTokenizer` 在运行时的行为。

### Lines 812-817: Method `MiMoAudioTokenizer.__init__`
```python
    def __init__(self, config: MiMoAudioTokenizerConfig):
        super().__init__(config)
        self.config = config
        self.sampling_rate = config.sampling_rate
        self.encoder = AudioEncoder(config=config)
        self.downsample_rate = int(config.hop_length * 2 * config.avg_pooler)
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 819-821: Method `MiMoAudioTokenizer.get_output_length`
```python
    def get_output_length(self, mel_len):
        tgt_len = mel_len + 3 - self.config.kernel_size
        return (tgt_len + 2 - self.config.kernel_size) // self.config.stride_size + 1
```
**EN:** This method implements `get_output_length(mel_len)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_output_length(mel_len)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 823-832: Method `MiMoAudioTokenizer.encode`
```python
    @torch.no_grad()
    def encode(self, mels, input_lens, use_quantizer=True):
        input_features = mels
        encoder_output_length = self.get_output_length(input_lens)
        hidden_states, hidden_states_packed, encoder_output_length, codes = (
            self.encoder.encode(
                input_features, input_lens=input_lens, use_quantizer=use_quantizer
            )
        )
        return hidden_states, hidden_states_packed, encoder_output_length, codes
```
**EN:** This method implements `encode(mels, input_lens, use_quantizer=...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoAudioTokenizer` Decorators: torch.no_grad().
**CN:** 这个方法实现了 `encode(mels, input_lens, use_quantizer=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoAudioTokenizer` 内部调用 装饰器：torch.no_grad()。

### Lines 835-864: Function `group_by_length`
```python
def group_by_length(features: torch.Tensor, lengths: torch.Tensor, max_length: int):
    if features.size(0) != lengths.sum().item():
        raise ValueError(
            f"Feature size mismatch: {features.size(0)} vs {lengths.sum().item()}"
        )

    split_points = []
    current_sum = 0

    for i, seq_len in enumerate(lengths):
        if current_sum + seq_len > max_length and current_sum > 0:
            split_points.append(i)
            current_sum = seq_len.item()
        else:
            current_sum += seq_len.item()

    # Convert split points to group sizes
    group_sizes = []
    prev = 0
    for point in split_points:
        group_sizes.append(point - prev)
        prev = point
    if prev < len(lengths):
        group_sizes.append(len(lengths) - prev)

    len_groups = torch.split(lengths, group_sizes)
    feature_sizes = [group.sum().item() for group in len_groups]
    feature_groups = torch.split(features, feature_sizes)

    return feature_groups, len_groups
```
**EN:** This function implements `group_by_length(features: ..., lengths: ..., max_length: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `group_by_length(features: ..., lengths: ..., max_length: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 867-883: Function `encode_batch`
```python
@torch.no_grad()
def encode_batch(
    audio_tokenizer_encoder,
    input_features: torch.Tensor,
    input_lens: torch.Tensor,
    max_length: int = 256000,
):
    feature_groups, len_groups = group_by_length(input_features, input_lens, max_length)

    encoded_parts = []
    for features, lengths in zip(feature_groups, len_groups):
        codes, _ = audio_tokenizer_encoder.encode(  # codes are also packed
            input_features=features, input_lens=lengths, return_codes_only=True
        )
        encoded_parts.append(codes)

    return torch.cat(encoded_parts, dim=-1)
```
**EN:** This function implements `encode_batch(audio_tokenizer_encoder, input_features: ..., input_lens: ..., max_length: ...=...)` and implements a focused helper that supports the surrounding runtime flow Decorators: torch.no_grad().
**CN:** 这个函数实现了 `encode_batch(audio_tokenizer_encoder, input_features: ..., input_lens: ..., max_length: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑 装饰器：torch.no_grad()。

### Lines 886-892: Function `_segment_lengths_for_mel`
```python
def _segment_lengths_for_mel(mel: torch.Tensor, segment_size: int):
    """Split mel into segments of segment_size with a possible shorter remainder."""
    input_len = mel.size(0)
    segs = [segment_size] * (input_len // segment_size)
    if input_len % segment_size > 0:
        segs.append(input_len % segment_size)
    return segs
```
**EN:** This function implements `_segment_lengths_for_mel(mel: ..., segment_size: ...)` and Split mel into segments of segment_size with a possible shorter remainder.
**CN:** 这个函数实现了 `_segment_lengths_for_mel(mel: ..., segment_size: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 895-927: Function `tokenize_audio_batch`
```python
@torch.no_grad()
def tokenize_audio_batch(mels, audio_tokenizer_encoder, segment_size=6000, device=None):
    """
    Tokenize multiple mels in one encode_batch call.
    Returns list of code tensors, each [T_i, C] for that mel.
    """
    if not mels:
        return []
    if device is None:
        device = next(audio_tokenizer_encoder.parameters()).device
    # Build segment lengths per mel
    input_len_seg_per_mel = [_segment_lengths_for_mel(m, segment_size) for m in mels]
    input_lens_flat = [s for segs in input_len_seg_per_mel for s in segs]
    input_features = torch.cat([m.to(device) for m in mels], dim=0)
    input_lens_t = torch.tensor(input_lens_flat, dtype=torch.long, device=device)
    codes_packed = encode_batch(
        audio_tokenizer_encoder,
        input_features=input_features,
        input_lens=input_lens_t,
    )
    codes = codes_packed.transpose(0, 1).detach()  # [total_code_T, C]
    # Code length per mel: must match encoder's actual output (get_output_length + optional avg_pooler downsampling)
    code_lengths = []
    for segs in input_len_seg_per_mel:
        out_len = audio_tokenizer_encoder.get_output_length(
            torch.tensor(segs, dtype=torch.long, device=device)
        )
        if getattr(audio_tokenizer_encoder, "down_sample_layer", None) is not None:
            avg = audio_tokenizer_encoder.config.avg_pooler
            out_len = out_len // avg + (out_len % avg != 0).long()
        code_lengths.append(out_len.sum().item())
    code_list = torch.split(codes, code_lengths)
    return list(code_list)
```
**EN:** This function implements `tokenize_audio_batch(mels, audio_tokenizer_encoder, segment_size=..., device=...)` and Tokenize multiple mels in one encode_batch call. Decorators: torch.no_grad().
**CN:** 这个函数实现了 `tokenize_audio_batch(mels, audio_tokenizer_encoder, segment_size=..., device=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.no_grad()。

### Lines 930-931: Class `MiMoAudioEncoderConfig` overview
```python
@dataclass
class MiMoAudioEncoderConfig:
```
**EN:** Defines `MiMoAudioEncoderConfig` as a reusable runtime type derived from no explicit base class. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoAudioEncoderConfig`，其继承关系为 no explicit base class。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 932-949: Class `MiMoAudioEncoderConfig` attributes
```python
    tokenizer_version: str = "v1"
    speech_vocab_size: str = "1025-1025-129-129-129-129-129-129"
    speech_zeroemb_idx: str = "1024-1024-128-128-128-128-128-128"
    group_size: int = 4
    audio_channels: int = 8
    input_local_layers: int = 6
    input_local_dim: int = 1024
    input_full_attention: bool = True
    input_local_attn_heads: int = 64
    input_local_head_dim: int = 16
    input_local_intermediate_size: int = 4096
    input_local_hidden_dropout: float = 0.0
    out_hidden_size: int = 4096  # mimo vl hidden dim
    rope_theta: float = 640000.0
    partial_rotary_factor: float = 0.334
    projection_layers: int = 1
    add_post_norm: bool = False
    audio_segment_size: int = 6000
```
**EN:** Defines class-level attributes and metadata that shape how `MiMoAudioEncoderConfig` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiMoAudioEncoderConfig` 在运行时的行为。

### Lines 952-952: Class `AudioProjection` overview
```python
class AudioProjection(nn.Module):
```
**EN:** Defines `AudioProjection` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AudioProjection`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 953-969: Method `AudioProjection.__init__`
```python
    def __init__(
        self,
        input_size,
        hidden_size,
        output_size,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.input_size = input_size
        self.hidden_size = hidden_size
        self.output_size = output_size
        self.mlp = nn.Sequential(
            nn.Linear(self.input_size, self.hidden_size, bias=False),
            nn.GELU(),
            nn.Linear(self.hidden_size, self.output_size, bias=False),
        )
```
**EN:** This method implements `__init__(input_size, hidden_size, output_size, quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(input_size, hidden_size, output_size, quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 971-972: Method `AudioProjection.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.mlp(x)
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 975-975: Class `MiMoV2AudioConfig` overview
```python
class MiMoV2AudioConfig:
```
**EN:** Defines `MiMoV2AudioConfig` as a reusable runtime type derived from no explicit base class. The class groups 9 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2AudioConfig`，其继承关系为 no explicit base class。这个类组织了 9 个方法，用于实现模型相关行为。

### Lines 976-1034: Method `MiMoV2AudioConfig.__init__`
```python
    def __init__(
        self,
        speech_vocab_size: str | int = "1280",
        speech_lm_head_sizes: str | int | None = None,
        speech_zeroemb_idx: str | int = "1280",
        delay_pattern: str = "0-1-2-3-4-5-6-7-7-7-7-7-7-7-7-7-7-7-7-7",
        group_size: int = 4,
        audio_channels: int = 20,
        input_local_dim: int = 1024,
        input_local_layers: int = 6,
        input_local_attn_heads: int = 16,
        input_local_intermediate_size: int = 4096,
        input_local_rope_theta: float = 640000.0,
        input_local_partial_rotary_factor: float = 1.0,
        output_local_dim: int = 1024,
        output_local_layers: int = 6,
        output_local_attn_heads: int = 16,
        output_local_intermediate_size: int = 4096,
        output_local_rope_theta: float = 640000.0,
        output_local_partial_rotary_factor: float = 1.0,
        input_projection_layers: int = 2,
        output_projection_layers: int = 2,
        add_encoder_post_norm: bool = True,
        audio_config: dict = None,
        **kwargs,
    ):
        for key, value in kwargs.items():
            setattr(self, key, value)

        if audio_config is not None:
            self._load_from_audio_config(audio_config)
        else:
            self.speech_vocab_size = speech_vocab_size
            self.speech_lm_head_sizes = (
                speech_lm_head_sizes
                if speech_lm_head_sizes is not None
                else speech_vocab_size
            )
            self.speech_zeroemb_idx = speech_zeroemb_idx
            self.delay_pattern = delay_pattern
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(speech_vocab_size: ...=..., speech_lm_head_sizes: ...=..., speech_zeroemb_idx: ...=..., delay_pattern: ...=..., group_size: ...=..., audio_channels: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(speech_vocab_size: ...=..., speech_lm_head_sizes: ...=..., speech_zeroemb_idx: ...=..., delay_pattern: ...=..., group_size: ...=..., audio_channels: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1036-1082: Method `MiMoV2AudioConfig._load_from_audio_config`
```python
    def _load_from_audio_config(self, audio_config: dict):
        """Load audio parameters from audio_config dict in checkpoint.

        Uses naming that matches megatron2hf conversion output to minimize manual mapping.
        """
        self.group_size = audio_config.get("group_size", 4)
        self.audio_channels = audio_config.get("audio_channels", 20)
        self.speech_vocab_size = audio_config.get("speech_vocab_size", "1280")
        self.speech_lm_head_sizes = audio_config.get(
            "speech_lm_head_sizes", self.speech_vocab_size
        )
        self.speech_zeroemb_idx = audio_config.get("speech_zeroemb_idx", "1280")
        # Per-channel decode delays; len must equal audio_channels.
        self.delay_pattern = audio_config.get(
            "audio_output_delay_pattern", "0-1-2-3-4-5-6-7-7-7-7-7-7-7-7-7-7-7-7-7"
        )

        self.input_local_dim = audio_config.get("input_local_dim", 1024)
        self.input_local_layers = audio_config.get("input_local_layers", 6)
        self.input_local_attn_heads = audio_config.get("input_local_attn_heads", 16)
        self.input_local_intermediate_size = audio_config.get(
            "input_local_intermediate_size", 4096
        )
        self.input_local_rope_theta = audio_config.get(
            "input_local_rope_theta", 640000.0
        )
        self.input_local_partial_rotary_factor = audio_config.get(
            "input_local_partial_rotary_factor", 1.0
        )

        self.output_local_dim = audio_config.get("output_local_dim", 1024)
        self.output_local_layers = audio_config.get("output_local_layers", 6)
        self.output_local_attn_heads = audio_config.get("output_local_attn_heads", 16)
        self.output_local_intermediate_size = audio_config.get(
            "output_local_intermediate_size", 4096
        )
        self.output_local_rope_theta = audio_config.get(
            "output_local_rope_theta", 640000.0
        )
        self.output_local_partial_rotary_factor = audio_config.get(
# ... truncated for brevity ...
```
**EN:** This method implements `_load_from_audio_config(audio_config: ...)` and Load audio parameters from audio_config dict in checkpoint.
**CN:** 这个方法实现了 `_load_from_audio_config(audio_config: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1084-1087: Method `MiMoV2AudioConfig._parse_maybe_list`
```python
    def _parse_maybe_list(self, value: str | int, length: int) -> list[int]:
        if isinstance(value, str) and "-" in value:
            return [int(s) for s in value.split("-")]
        return [int(value)] * length
```
**EN:** This method implements `_parse_maybe_list(value: ..., length: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2AudioConfig`.
**CN:** 这个方法实现了 `_parse_maybe_list(value: ..., length: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2AudioConfig` 内部调用。

### Lines 1089-1090: Method `MiMoV2AudioConfig.parsed_speech_empty_ids`
```python
    def parsed_speech_empty_ids(self):
        return self._parse_maybe_list(self.speech_zeroemb_idx, self.audio_channels)
```
**EN:** This method implements `parsed_speech_empty_ids()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2AudioConfig`.
**CN:** 这个方法实现了 `parsed_speech_empty_ids()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2AudioConfig` 内部调用。

### Lines 1092-1093: Method `MiMoV2AudioConfig.parsed_speech_vocab_sizes`
```python
    def parsed_speech_vocab_sizes(self):
        return self._parse_maybe_list(self.speech_vocab_size, self.audio_channels)
```
**EN:** This method implements `parsed_speech_vocab_sizes()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2AudioConfig`.
**CN:** 这个方法实现了 `parsed_speech_vocab_sizes()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2AudioConfig` 内部调用。

### Lines 1095-1096: Method `MiMoV2AudioConfig.parsed_speech_lm_head_sizes`
```python
    def parsed_speech_lm_head_sizes(self):
        return self._parse_maybe_list(self.speech_lm_head_sizes, self.audio_channels)
```
**EN:** This method implements `parsed_speech_lm_head_sizes()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2AudioConfig`.
**CN:** 这个方法实现了 `parsed_speech_lm_head_sizes()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2AudioConfig` 内部调用。

### Lines 1098-1099: Method `MiMoV2AudioConfig.parsed_delay_pattern`
```python
    def parsed_delay_pattern(self):
        return self._parse_maybe_list(self.delay_pattern, self.audio_channels)
```
**EN:** This method implements `parsed_delay_pattern()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2AudioConfig`.
**CN:** 这个方法实现了 `parsed_delay_pattern()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2AudioConfig` 内部调用。

### Lines 1101-1122: Method `MiMoV2AudioConfig.input_local_config`
```python
    def input_local_config(self):
        """Create config for input local transformer."""
        config = Qwen2Config()
        for attr in dir(self):
            if not attr.startswith("_") and hasattr(config, attr):
                setattr(config, attr, getattr(self, attr))

        config.hidden_size = self.input_local_dim
        config.num_hidden_layers = self.input_local_layers
        config.num_attention_heads = self.input_local_attn_heads
        config.num_key_value_heads = self.input_local_attn_heads
        config.head_dim = getattr(
            self,
            "input_local_head_dim",
            self.input_local_dim // self.input_local_attn_heads,
        )
        config.intermediate_size = self.input_local_intermediate_size
        config.rope_theta = self.input_local_rope_theta
        config.partial_rotary_factor = self.input_local_partial_rotary_factor
        config._attn_implementation_internal = "sdpa"

        return config
```
**EN:** This method implements `input_local_config()` and Create config for input local transformer.
**CN:** 这个方法实现了 `input_local_config()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1124-1141: Method `MiMoV2AudioConfig.output_local_config`
```python
    def output_local_config(self):
        """Create config for output local transformer."""
        config = Qwen2Config()
        for attr in dir(self):
            if not attr.startswith("_") and hasattr(config, attr):
                setattr(config, attr, getattr(self, attr))

        config.hidden_size = self.output_local_dim
        config.num_hidden_layers = self.output_local_layers
        config.num_attention_heads = self.output_local_attn_heads
        config.num_key_value_heads = self.output_local_attn_heads
        config.head_dim = self.output_local_dim // self.output_local_attn_heads
        config.intermediate_size = self.output_local_intermediate_size
        config.rope_theta = self.output_local_rope_theta
        config.partial_rotary_factor = self.output_local_partial_rotary_factor
        config._attn_implementation_internal = "sdpa"

        return config
```
**EN:** This method implements `output_local_config()` and Create config for output local transformer.
**CN:** 这个方法实现了 `output_local_config()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1144-1144: Class `MiMoAudioEncoder` overview
```python
class MiMoAudioEncoder(nn.Module):
```
**EN:** Defines `MiMoAudioEncoder` as a reusable runtime type derived from nn.Module. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoAudioEncoder`，其继承关系为 nn.Module。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 1145-1145: Class `MiMoAudioEncoder` attributes
```python
    config: MiMoAudioEncoderConfig
```
**EN:** Defines class-level attributes and metadata that shape how `MiMoAudioEncoder` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiMoAudioEncoder` 在运行时的行为。

### Lines 1147-1219: Method `MiMoAudioEncoder.__init__`
```python
    def __init__(self, config):
        super().__init__()
        if not isinstance(config, MiMoV2AudioConfig):
            config_dict = (
                vars(config) if hasattr(config, "__dict__") else config.__dict__
            )
            config = MiMoV2AudioConfig(**config_dict)
        self.config = config
        self.server_args = get_global_server_args()
        self.use_data_parallel = get_global_server_args().mm_enable_dp_encoder
        self.speech_empty_ids = self.parsed_speech_empty_ids()
        self.audio_channels = config.audio_channels
        self.audio_group_size = config.group_size
        self.audio_segment_size = config.audio_segment_size
        speech_vocab_size = self._parse_maybe_list(
            self.config.speech_vocab_size, self.config.audio_channels
        )
        input_local_config = Qwen2Config(
            hidden_size=self.config.input_local_dim,
            num_hidden_layers=self.config.input_local_layers,
            num_attention_heads=self.config.input_local_attn_heads,
            num_key_value_heads=self.config.input_local_attn_heads,
            intermediate_size=self.config.input_local_intermediate_size,
            attention_dropout=self.config.input_local_hidden_dropout,
            rope_theta=self.config.rope_theta,
            partial_rotary_factor=self.config.partial_rotary_factor,
        )
        input_local_config.head_dim = self.config.input_local_head_dim

        self.input_local_transformer = Qwen2Model(input_local_config)

        if not self.config.add_post_norm:
            self.input_local_transformer.norm = nn.Identity()

        self.speech_embeddings = nn.ModuleList(
            [
                nn.Embedding(
                    speech_vocab_size[i],
                    self.config.input_local_dim,
                    padding_idx=self.speech_empty_ids[i],
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1221-1250: Method `MiMoAudioEncoder._load_audio_tokenizer`
```python
    @staticmethod
    def _load_audio_tokenizer(path: str, device: torch.device) -> MiMoAudioTokenizer:
        """Load MiMoAudioTokenizer manually to avoid new-transformers compat issues."""
        import json
        import os

        from safetensors.torch import load_file

        config_path = os.path.join(path, "config.json")
        with open(config_path) as f:
            config_dict = json.load(f)
        config = MiMoAudioTokenizer.config_class(**config_dict)
        model = MiMoAudioTokenizer(config)
        # Load weights from safetensors or pytorch bin
        safetensors_path = os.path.join(path, "model.safetensors")
        bin_path = os.path.join(path, "pytorch_model.bin")
        if os.path.exists(safetensors_path):
            state_dict = load_file(safetensors_path, device="cpu")
        elif os.path.exists(bin_path):
            state_dict = torch.load(bin_path, map_location="cpu", weights_only=True)
        else:
            raise FileNotFoundError(
                f"No model weights found in {path} "
                "(expected model.safetensors or pytorch_model.bin)"
            )
        model.load_state_dict(state_dict, strict=False)
        model = model.to(device=device, dtype=torch.bfloat16)
        model.eval()
        model.requires_grad_(False)
        return model
```
**EN:** This method implements `_load_audio_tokenizer(device: ...)` and Load MiMoAudioTokenizer manually to avoid new-transformers compat issues. Decorators: staticmethod.
**CN:** 这个方法实现了 `_load_audio_tokenizer(device: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 1252-1255: Method `MiMoAudioEncoder.parsed_speech_empty_ids`
```python
    def parsed_speech_empty_ids(self):
        return self._parse_maybe_list(
            self.config.speech_zeroemb_idx, self.config.audio_channels
        )
```
**EN:** This method implements `parsed_speech_empty_ids()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoAudioEncoder`.
**CN:** 这个方法实现了 `parsed_speech_empty_ids()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoAudioEncoder` 内部调用。

### Lines 1257-1260: Method `MiMoAudioEncoder._parse_maybe_list`
```python
    def _parse_maybe_list(self, value: str | int, length: int) -> List[int]:
        if isinstance(value, str) and "-" in value:
            return [int(s) for s in value.split("-")]
        return [int(value)] * length
```
**EN:** This method implements `_parse_maybe_list(value: ..., length: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoAudioEncoder`.
**CN:** 这个方法实现了 `_parse_maybe_list(value: ..., length: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoAudioEncoder` 内部调用。

### Lines 1263-1269: Method `MiMoAudioEncoder.apply_input_local_transformer`
```python
    def apply_input_local_transformer(self, speech_embeddings: torch.Tensor):
        output = self.input_local_transformer(
            inputs_embeds=speech_embeddings,
            return_dict=True,
            is_causal=not self.config.input_full_attention,  # for SDPA
        )
        return output.last_hidden_state  # [T//group_size, group_size, input_local_dim]
```
**EN:** This method implements `apply_input_local_transformer(speech_embeddings: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoAudioEncoder`.
**CN:** 这个方法实现了 `apply_input_local_transformer(speech_embeddings: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoAudioEncoder` 内部调用。

### Lines 1271-1280: Method `MiMoAudioEncoder.apply_speech_embeddings`
```python
    def apply_speech_embeddings(self, audio_codes: torch.Tensor) -> torch.Tensor:
        num_segments = audio_codes.shape[0]
        _audio_embeddings = torch.zeros(
            (num_segments, self.config.group_size, self.config.input_local_dim),
            dtype=next(self.speech_embeddings[0].parameters()).dtype,
            device=audio_codes.device,
        )
        for i in range(self.config.audio_channels):
            _audio_embeddings.add_(self.speech_embeddings[i](audio_codes[:, :, i]))
        return _audio_embeddings
```
**EN:** This method implements `apply_speech_embeddings(audio_codes: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `apply_speech_embeddings(audio_codes: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1282-1308: Method `MiMoAudioEncoder.process_audio`
```python
    def process_audio(self, audio):
        T = audio.shape[0]
        audio = audio[:, : self.audio_channels]
        padded_T = (
            (T + self.audio_group_size - 1)
            // self.audio_group_size
            * self.audio_group_size
        )
        padded_audio = torch.cat(
            [
                audio,
                torch.zeros(
                    padded_T - T,
                    self.audio_channels,
                    dtype=torch.int32,
                    device=audio.device,
                )
                + audio[-1, :],
            ],
            dim=0,
        )  # pad using the last embedding
        padded_audio = padded_audio.reshape(
            padded_T // self.audio_group_size,
            self.audio_group_size,
            self.audio_channels,
        )
        return padded_audio
```
**EN:** This method implements `process_audio(audio)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoAudioEncoder`.
**CN:** 这个方法实现了 `process_audio(audio)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoAudioEncoder` 内部调用。

### Lines 1310-1350: Method `MiMoAudioEncoder.get_audio_feature`
```python
    def get_audio_feature(self, items) -> torch.Tensor:
        # items: already audio-only MultimodalDataItem list from caller.
        # Each item.feature is either one mel tensor or a list of mel tensors (e.g. long audio split into chunks).
        all_mels = []
        for item in items:
            f = item.feature
            if isinstance(f, (list, tuple)):
                all_mels.extend(f)
            else:
                all_mels.append(f)
        if not all_mels:
            device = next(self.projection.parameters()).device
            dtype = next(self.projection.parameters()).dtype
            return torch.empty(
                0, self.config.out_hidden_size, device=device, dtype=dtype
            )
        # Batch tokenize: one encode_batch call for all mels
        device = next(self.audio_tokenizer.encoder.parameters()).device
        code_list = tokenize_audio_batch(
            all_mels,
            self.audio_tokenizer.encoder,
            segment_size=self.audio_segment_size,
            device=device,
        )
        codecs_to_concat = []
        for codecs in code_list:
            padded_codes = self.process_audio(
                codecs
            )  # [T//group_size, group_size, audio_channels]
            codecs_to_concat.append(padded_codes)
        audio_codes = torch.cat(
            codecs_to_concat, dim=0
        )  # [T//group_size, group_size, audio_channels]

        _audio_embeddings = self.apply_speech_embeddings(audio_codes)
        audio_embeds = self.apply_input_local_transformer(
            _audio_embeddings
        )  #  [T//group_size,  group_size, input_local_dim]
        B = audio_embeds.shape[0]
        audio_embeds = self.projection(audio_embeds.reshape(B, -1))
# ... truncated for brevity ...
```
**EN:** This method implements `get_audio_feature(items)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_audio_feature(items)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `logging`
- `math`
- `os`
- `typing`
- `dataclasses.dataclass`
- `functools.wraps`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `einops.rearrange`
- `transformers.activations.ACT2FN`
- `transformers.configuration_utils.PretrainedConfig`
- `transformers.modeling_utils.PreTrainedModel`
- `transformers.models.qwen2.configuration_qwen2.Qwen2Config`
- `transformers.models.qwen2.modeling_qwen2.Qwen2Model`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.is_cuda`
- `sgl_kernel.flash_attn.flash_attn_varlen_func`
- `json`
- `safetensors.torch.load_file`
- `huggingface_hub.snapshot_download`
