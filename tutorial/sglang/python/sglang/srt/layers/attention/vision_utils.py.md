# vision_utils.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/vision_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides vision utils utilities shared by multiple attention implementations in SGLang. / 该模块提供 SGLang 多种注意力实现共享的 vision utils 工具函数。
## Line-by-Line Analysis / 逐行分析
### Lines 1-1: docstring
```python
"""Utility functions for vision attention layers."""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 3-5: imports
```python
import torch

from sglang.srt.layers.dp_attention import get_attention_tp_size
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 8-23: function update_vit_attn_dummy_heads_config
```python
def update_vit_attn_dummy_heads_config(config):
    """Update HF config to ensure vision attention num_attention_heads is divisible by tp_size"""
    tp_size = get_attention_tp_size()
    num_heads = getattr(
        config.vision_config,
        "num_heads",
        getattr(config.vision_config, "num_attention_heads", None),
    )
    head_dim = config.vision_config.hidden_size // num_heads
    num_dummy_heads = 0

    if num_heads % tp_size != 0:
        num_dummy_heads = ((num_heads + tp_size - 1) // tp_size) * tp_size - num_heads

    setattr(config.vision_config, "head_dim", head_dim)
    setattr(config.vision_config, "num_dummy_heads", num_dummy_heads)
```
**EN:** Updates update vit attn dummy heads config on the active object so later attention steps observe the latest runtime state.
**CN:** 更新活动对象上的 update vit attn dummy heads config，以便后续注意力步骤读取最新运行时状态。

### Lines 26-65: function pad_vit_attn_dummy_heads
```python
def pad_vit_attn_dummy_heads(config, name: str, loaded_weight: torch.Tensor):
    """Pad attention qkv weights for dummy heads"""
    num_dummy_heads = config.vision_config.num_dummy_heads
    if num_dummy_heads == 0:
        return loaded_weight
    head_dim = config.vision_config.head_dim

    if "attn.qkv_proj" in name:
        wq, wk, wv = loaded_weight.chunk(3, dim=0)
        if name.endswith(".weight"):
            dummy_shape = [num_dummy_heads, head_dim, wq.shape[-1]]
        elif name.endswith(".bias"):
            dummy_shape = [num_dummy_heads, head_dim]
        else:
            raise RuntimeError(f"Unsupported weight with name={name}")
        pad_func = lambda x: torch.cat(
            [x.unflatten(0, (-1, head_dim)), x.new_zeros(dummy_shape)], dim=0
        ).flatten(0, 1)
        wq, wk, wv = pad_func(wq), pad_func(wk), pad_func(wv)
        loaded_weight = torch.cat([wq, wk, wv], dim=0)
    elif any([_ in name for _ in ["attn.q_proj", "attn.k_proj", "attn.v_proj"]]):
        if name.endswith(".weight"):
            dummy_shape = [num_dummy_heads, head_dim, loaded_weight.shape[-1]]
        elif name.endswith(".bias"):
            dummy_shape = [num_dummy_heads, head_dim]
        else:
            raise RuntimeError(f"Unsupported weight with name={name}")
        padded_weight = loaded_weight.new_zeros(dummy_shape)
        loaded_weight = torch.cat(
            [loaded_weight.unflatten(0, (-1, head_dim)), padded_weight], dim=0
        ).flatten(0, 1)
    elif "attn.proj.weight" in name:
        padded_weight = loaded_weight.new_zeros(
            loaded_weight.shape[0], head_dim * num_dummy_heads
        )
        loaded_weight = torch.cat([loaded_weight, padded_weight], dim=-1)
    elif "attn.q_norm.weight" in name or "attn.k_norm.weight" in name:
        padded_weight = loaded_weight.new_zeros(head_dim * num_dummy_heads)
        loaded_weight = torch.cat([loaded_weight, padded_weight], dim=0)
    return loaded_weight
```
**EN:** Implements the pad vit attn dummy heads routine used by this attention module.
**CN:** 实现该注意力模块使用的 pad vit attn dummy heads 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排

## Dependencies / 依赖关系
- `torch`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
