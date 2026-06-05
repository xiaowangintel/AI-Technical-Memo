# zimage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/zimage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for zimage within the multimodal runtime. Key symbols include `SelectFirstElement`, `TimestepEmbedder`, `FeedForward`. / 该模块实现多模态运行时中与 zimage 相关的模型构件。 关键符号包括 `SelectFirstElement`, `TimestepEmbedder`, `FeedForward`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Imports and module setup / 导入与模块初始化
```python
import math
from typing import Any, List, Optional, Tuple

import torch
import torch.nn as nn

from sglang.multimodal_gen.configs.models.dits.zimage import ZImageDitConfig
from sglang.multimodal_gen.runtime.distributed import (
    get_sp_parallel_rank,
    get_sp_world_size,
    get_tp_world_size,
    sequence_model_parallel_all_gather,
)
from sglang.multimodal_gen.runtime.distributed.parallel_state import (
# ...
_is_cuda = current_platform.is_cuda()

ADALN_EMBED_DIM = 256
SEQ_MULTI_OF = 32
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 63-68: Class `SelectFirstElement` / 类 `SelectFirstElement`
```python
class SelectFirstElement(nn.Module):
    def __init__(self):
        super().__init__()

    def forward(self, x):
        return x[0]
```
**EN:** This class models `SelectFirstElement` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `SelectFirstElement`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 71-115: Class `TimestepEmbedder` / 类 `TimestepEmbedder`
```python
class TimestepEmbedder(nn.Module):
    def __init__(self, out_size, mid_size=None, frequency_embedding_size=256):
        super().__init__()
        if mid_size is None:
            mid_size = out_size

        self.mlp = nn.ModuleList(
            [
                ColumnParallelLinear(
                    frequency_embedding_size, mid_size, bias=True, gather_output=False
                ),
                nn.SiLU(),
                RowParallelLinear(
                    mid_size, out_size, bias=True, input_is_parallel=True
# ...
        t_emb, _ = self.mlp[0](t_freq)
        t_emb = self.mlp[1](t_emb)
        t_emb, _ = self.mlp[2](t_emb)
        return t_emb
```
**EN:** This class models `TimestepEmbedder` as a specialization of `nn.Module`. Important methods include `__init__`, `timestep_embedding`, `forward`.
**CN:** 该类实现 `TimestepEmbedder`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `timestep_embedding`, `forward`。

### Lines 118-150: Class `FeedForward` / 类 `FeedForward`
```python
class FeedForward(nn.Module):
    def __init__(
        self,
        dim: int,
        hidden_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        # Use MergedColumnParallelLinear for gate and up projection (fused)
        self.w13 = MergedColumnParallelLinear(
            dim,
            [hidden_dim, hidden_dim],
            bias=False,
# ...
        x13, _ = self.w13(x)
        x = self.act(x13)
        out, _ = self.w2(x)
        return out
```
**EN:** This class models `FeedForward` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `FeedForward`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 153-372: Class `ZImageAttention` / 类 `ZImageAttention`
```python
class ZImageAttention(nn.Module):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        num_kv_heads: int,
        qk_norm: bool = True,
        eps: float = 1e-6,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.dim = dim
        self.head_dim = dim // num_heads
# ...

        hidden_states, _ = self.to_out[0](hidden_states)

        return hidden_states
```
**EN:** This class models `ZImageAttention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ZImageAttention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 375-529: Class `ZImageTransformerBlock` / 类 `ZImageTransformerBlock`
```python
class ZImageTransformerBlock(nn.Module):
    def __init__(
        self,
        layer_id: int,
        dim: int,
        n_heads: int,
        n_kv_heads: int,
        norm_eps: float,
        qk_norm: bool,
        modulation=True,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
# ...
            )
            x = x + self.ffn_norm2(ffn_out)

        return x
```
**EN:** This class models `ZImageTransformerBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ZImageTransformerBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 532-551: Class `FinalLayer` / 类 `FinalLayer`
```python
class FinalLayer(nn.Module):
    def __init__(self, hidden_size, out_channels):
        super().__init__()
        self.norm_final = nn.LayerNorm(hidden_size, elementwise_affine=False, eps=1e-6)
        self.linear = ColumnParallelLinear(
            hidden_size, out_channels, bias=True, gather_output=True
        )

        self.act = nn.SiLU()
        self.adaLN_modulation = nn.Sequential(
            nn.SiLU(),
            ReplicatedLinear(min(hidden_size, ADALN_EMBED_DIM), hidden_size, bias=True),
        )

# ...
        scale = 1.0 + scale
        x = self.norm_final(x) * scale.unsqueeze(1)
        x, _ = self.linear(x)
        return x
```
**EN:** This class models `FinalLayer` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `FinalLayer`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 554-619: Class `RopeEmbedder` / 类 `RopeEmbedder`
```python
class RopeEmbedder:
    def __init__(
        self,
        theta: float = 256.0,
        axes_dims: List[int] = (16, 56, 56),
        axes_lens: List[int] = (64, 128, 128),
    ):
        self.theta = theta
        self.axes_dims = axes_dims
        self.axes_lens = axes_lens
        assert len(axes_dims) == len(
            axes_lens
        ), "axes_dims and axes_lens must have the same length"

# ...
            cos_out.append(self.cos_cached[i][index])
            sin_out.append(self.sin_cached[i][index])

        return torch.cat(cos_out, dim=-1), torch.cat(sin_out, dim=-1)
```
**EN:** This class models `RopeEmbedder`. Important methods include `__init__`, `precompute_freqs`, `__call__`.
**CN:** 该类实现 `RopeEmbedder`。 其中较重要的方法包括 `__init__`, `precompute_freqs`, `__call__`。

### Lines 622-1033: Class `ZImageTransformer2DModel` / 类 `ZImageTransformer2DModel`
```python
class ZImageTransformer2DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    _supports_gradient_checkpointing = True
    _no_split_modules = ["ZImageTransformerBlock"]
    _fsdp_shard_conditions = ZImageDitConfig().arch_config._fsdp_shard_conditions
    param_names_mapping = ZImageDitConfig().arch_config.param_names_mapping

    param_names_mapping = ZImageDitConfig().arch_config.param_names_mapping
    reverse_param_names_mapping = (
        ZImageDitConfig().arch_config.reverse_param_names_mapping
    )

    # Maps fused runtime layer names to their checkpoint shard names.
    # Used by is_layer_skipped() to correctly handle --quantization-ignored-layers
    # Only list fusions that are unconditional. Conditional fusions (e.g. to_qkv for
# ...
        x = self.unpatchify(x, x_size, patch_size, f_patch_size)

        # Keep batch dim so output shape matches input (e.g. rollout/scheduler expect same ndim).
        return -torch.stack(x)
```
**EN:** This class models `ZImageTransformer2DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. Important methods include `get_nunchaku_quant_rules`, `__init__`, `unpatchify`, `create_coordinate_grid`.
**CN:** 该类实现 `ZImageTransformer2DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `get_nunchaku_quant_rules`, `__init__`, `unpatchify`, `create_coordinate_grid`。

### Lines 1034-1036: Top-level configuration / 顶层配置
```python


EntryClass = ZImageTransformer2DModel
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.zimage`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`
- **External / 外部**: `torch`, `torch.nn`, `nunchaku.models.attention`, `diffusers`
- **Stdlib / 标准库**: `math`, `typing`
