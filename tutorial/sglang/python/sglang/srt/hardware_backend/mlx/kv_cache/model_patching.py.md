# model_patching.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/kv_cache/model_patching.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for KV-cache management inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的KV Cache 管理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module setup and shared state / 模块设置与共享状态
```python
"""Model introspection and attention patching."""

from typing import Any

from sglang.srt.hardware_backend.mlx.kv_cache.attention_wrapper import (
    MLXAttentionWrapper,
)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `sglang.srt.hardware_backend.mlx.kv_cache.attention_wrapper`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `sglang.srt.hardware_backend.mlx.kv_cache.attention_wrapper`。

### Lines 10-23: Function `find_attention_layers` / 函数 `find_attention_layers`
```python
def find_attention_layers(model: Any) -> tuple[list[Any], str]:
    """Find transformer layers and the attention attribute name."""
    root = getattr(model, "language_model", model)
    container = getattr(root, "model", root)
    layer_list = getattr(container, "layers", None) or getattr(root, "layers", [])

    if layer_list:
        sample = layer_list[0]
        if hasattr(sample, "self_attn"):
            return layer_list, "self_attn"
        if hasattr(sample, "attention"):
            return layer_list, "attention"
        raise ValueError(f"No attention attribute in layer type {type(sample)}")
    return layer_list, "self_attn"
```
**EN:** This function implements `find_attention_layers`. It primarily calls `getattr`, `hasattr`, `ValueError`, `type` to complete its work. State updates are written into `root`, `container`, `layer_list`, `sample`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `find_attention_layers`。 它主要通过调用 `getattr`, `hasattr`, `ValueError`, `type` 来完成任务。 状态更新主要写入 `root`, `container`, `layer_list`, `sample`。 实现中使用了条件分支。

### Lines 26-40: Function `patch_model_attention` / 函数 `patch_model_attention`
```python
def patch_model_attention(model: Any) -> int:
    """Install MLXAttentionWrapper on all attention layers (idempotent).

    The wrapper delegates to the inner module when no BatchedDecodeContext
    is set, so it is always installed and never removed.
    """
    layer_list, attn_attr = find_attention_layers(model)
    patched = 0
    for idx, layer in enumerate(layer_list):
        attn = getattr(layer, attn_attr)
        if isinstance(attn, MLXAttentionWrapper):
            continue
        setattr(layer, attn_attr, MLXAttentionWrapper(attn, idx))
        patched += 1
    return patched
```
**EN:** This function implements `patch_model_attention`. It primarily calls `find_attention_layers`, `enumerate`, `getattr`, `isinstance`, `setattr`, `MLXAttentionWrapper` to complete its work. State updates are written into `patched`, `attn`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `patch_model_attention`。 它主要通过调用 `find_attention_layers`, `enumerate`, `getattr`, `isinstance`, `setattr`, `MLXAttentionWrapper` 来完成任务。 状态更新主要写入 `patched`, `attn`。 实现中使用了条件分支、迭代逻辑。

### Lines 43-46: Function `get_num_layers` / 函数 `get_num_layers`
```python
def get_num_layers(model: Any) -> int:
    """Return the number of transformer layers."""
    layer_list, _ = find_attention_layers(model)
    return len(layer_list)
```
**EN:** This function implements `get_num_layers`. It primarily calls `find_attention_layers`, `len` to complete its work.
**CN:** 该函数实现了 `get_num_layers`。 它主要通过调用 `find_attention_layers`, `len` 来完成任务。

## Key Concepts / 关键概念
- **Functions / 函数**: `find_attention_layers`, `patch_model_attention`, `get_num_layers`
- **Themes / 主题**: `kv_cache`, `patch`, `cache`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.mlx.kv_cache.attention_wrapper`
- **External / 外部依赖**: None / 无
- **Standard library / 标准库**: `typing`
