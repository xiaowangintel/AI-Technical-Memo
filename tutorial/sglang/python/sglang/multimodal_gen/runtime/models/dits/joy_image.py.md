# joy_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/dits/joy_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for joy image within the multimodal runtime. Key symbols include `fused_add_gate`, `ModulateWan`, `MMDoubleStreamBlock`. / 该模块实现多模态运行时中与 joy image 相关的模型构件。 关键符号包括 `fused_add_gate`, `ModulateWan`, `MMDoubleStreamBlock`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0

import math
from functools import lru_cache
from typing import Any, Optional, Tuple

import torch
import torch.nn as nn
from einops import rearrange

from sglang.multimodal_gen.configs.models.dits.joy_image import JoyImageDiTConfig
from sglang.multimodal_gen.runtime.distributed import (
    get_sp_group,
    get_sp_world_size,
# ...
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
_MODULATION_FACTOR = 6
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 45-63: Function `fused_add_gate` / 函数 `fused_add_gate`
```python
def fused_add_gate(
    residual: torch.Tensor, x: torch.Tensor, gate: torch.Tensor
) -> torch.Tensor:
    """Fused residual addition with gate.

    Computes: residual + x * gate.unsqueeze(1)

    This fuses the gate multiplication and residual addition to reduce
    intermediate tensor allocations and memory bandwidth.

    Args:
        residual (torch.Tensor): The residual tensor to add to. Shape: (B, L, D)
        x (torch.Tensor): The input tensor to be gated. Shape: (B, L, D)
        gate (torch.Tensor): The gate tensor. Shape: (B, D)

    Returns:
        torch.Tensor: residual + x * gate.unsqueeze(1)
    """
    return torch.addcmul(residual, x, gate.unsqueeze(1))
```
**EN:** This function drives `fused_add_gate` with inputs such as `residual`, `x`, `gate`. Fused residual addition with gate.
**CN:** 这个函数负责 `fused_add_gate`，主要处理 `residual`, `x`, `gate` 等输入。 文档字符串说明：Fused residual addition with gate.

### Lines 66-90: Class `ModulateWan` / 类 `ModulateWan`
```python
class ModulateWan(nn.Module):
    """Modulation layer for WanX."""

    def __init__(self, hidden_size: int, factor: int, dtype=None, device=None):
        super().__init__()
        self.factor = factor
        self.modulate_table = nn.Parameter(
            torch.zeros(1, factor, hidden_size, dtype=dtype, device=device)
            / hidden_size**0.5,
            requires_grad=False,
        )
        set_weight_attrs(
            self.modulate_table,
            {
# ...
            x = x.unsqueeze(1)
        return [
            o.squeeze(1) for o in (self.modulate_table + x).chunk(self.factor, dim=1)
        ]
```
**EN:** This class models `ModulateWan` as a specialization of `nn.Module`. Modulation layer for WanX. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `ModulateWan`，并继承/扩展 `nn.Module`。 文档字符串指出：Modulation layer for WanX. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 93-329: Class `MMDoubleStreamBlock` / 类 `MMDoubleStreamBlock`
```python
class MMDoubleStreamBlock(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        heads_num: int,
        mlp_width_ratio: float,
        mlp_act_type: str = "gelu_pytorch_tanh",
        supported_attention_backends: set[AttentionBackendEnum] | None = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.heads_num = heads_num
        self.hidden_size = hidden_size
# ...
            txt_mod2_gate,
        )

        return img, txt
```
**EN:** This class models `MMDoubleStreamBlock` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `MMDoubleStreamBlock`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 332-575: Class `JoyTransformer3DModel` / 类 `JoyTransformer3DModel`
```python
class JoyTransformer3DModel(CachableDiT, LayerwiseOffloadableModuleMixin):
    """
    JoyImage Transformer 3D Model for image generation.

    """

    _supports_gradient_checkpointing = True
    _fsdp_shard_conditions = JoyImageDiTConfig()._fsdp_shard_conditions
    _compile_conditions = JoyImageDiTConfig()._compile_conditions
    _supported_attention_backends = JoyImageDiTConfig()._supported_attention_backends
    param_names_mapping = JoyImageDiTConfig().param_names_mapping
    reverse_param_names_mapping = JoyImageDiTConfig().reverse_param_names_mapping
    lora_param_names_mapping = JoyImageDiTConfig().lora_param_names_mapping

# ...
            c=self.out_channels,
        )

        return img
```
**EN:** This class models `JoyTransformer3DModel` as a specialization of `CachableDiT`, `LayerwiseOffloadableModuleMixin`. JoyImage Transformer 3D Model for image generation. Important methods include `__init__`, `_compute_rope_for_local_shard`, `forward`.
**CN:** 该类实现 `JoyTransformer3DModel`，并继承/扩展 `CachableDiT`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：JoyImage Transformer 3D Model for image generation. 其中较重要的方法包括 `__init__`, `_compute_rope_for_local_shard`, `forward`。

### Lines 578-581: Class `JoyImageEditTransformer3DModel` / 类 `JoyImageEditTransformer3DModel`
```python
class JoyImageEditTransformer3DModel(JoyTransformer3DModel):
    """Backward-compatible alias for JoyImageEdit model configs."""

    pass
```
**EN:** This class models `JoyImageEditTransformer3DModel` as a specialization of `JoyTransformer3DModel`. Backward-compatible alias for JoyImageEdit model configs.
**CN:** 该类实现 `JoyImageEditTransformer3DModel`，并继承/扩展 `JoyTransformer3DModel`。 文档字符串指出：Backward-compatible alias for JoyImageEdit model configs.

### Lines 582-584: Top-level configuration / 顶层配置
```python


EntryClass = [JoyTransformer3DModel, JoyImageEditTransformer3DModel]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.dits.joy_image`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.mlp`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`
- **External / 外部**: `torch`, `torch.nn`, `einops`
- **Stdlib / 标准库**: `math`, `functools`, `typing`
