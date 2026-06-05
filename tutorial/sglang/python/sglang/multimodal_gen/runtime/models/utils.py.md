# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects shared utility helpers that are reused across the multimodal generation stack. Key symbols include `set_weight_attrs`, `_make_synced_weight_loader`, `extract_layer_index`. / 该模块汇总了多模态生成栈中可复用的通用工具函数。 关键符号包括 `set_weight_attrs`, `_make_synced_weight_loader`, `extract_layer_index`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/model_executor/utils.py
"""Utils for model executor."""

from typing import Any

import torch

from sglang.srt.utils import (
    get_bool_env_var,
    is_gfx95_supported,
# ...
_is_hip = is_hip()
_is_gfx95_supported = is_gfx95_supported()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
_use_aiter_gfx95 = _use_aiter and _is_gfx95_supported
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 24-55: Function `set_weight_attrs` / 函数 `set_weight_attrs`
```python
def set_weight_attrs(
    weight: torch.Tensor,
    weight_attrs: dict[str, Any] | None,
):
    """Set attributes on a weight tensor.

    This method is used to set attributes on a weight tensor. This method
    will not overwrite existing attributes.

    Args:
        weight: The weight tensor.
        weight_attrs: A dictionary of attributes to set on the weight tensor.
    """
    if weight_attrs is None:
# ...

        if current_platform.is_tpu() and key == "weight_loader":
            value = _make_synced_weight_loader(value)
        setattr(weight, key, value)
```
**EN:** This function drives `set_weight_attrs` with inputs such as `weight`, `weight_attrs`. Set attributes on a weight tensor.
**CN:** 这个函数负责 `set_weight_attrs`，主要处理 `weight`, `weight_attrs` 等输入。 文档字符串说明：Set attributes on a weight tensor.

### Lines 58-64: Function `_make_synced_weight_loader` / 函数 `_make_synced_weight_loader`
```python
def _make_synced_weight_loader(original_weight_loader) -> Any:

    def _synced_weight_loader(param, *args, **kwargs):
        original_weight_loader(param, *args, **kwargs)
        torch._sync(param)

    return _synced_weight_loader
```
**EN:** This function drives `_make_synced_weight_loader` with inputs such as `original_weight_loader`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_make_synced_weight_loader`，主要处理 `original_weight_loader` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 67-86: Function `extract_layer_index` / 函数 `extract_layer_index`
```python
def extract_layer_index(layer_name: str) -> int:
    """
    Extract the layer index from the module name.
    Examples:
    - "encoder.layers.0" -> 0
    - "encoder.layers.1.self_attn" -> 1
    - "2.self_attn" -> 2
    - "model.encoder.layers.0.sub.1" -> ValueError
    """
    subnames = layer_name.split(".")
    int_vals: list[int] = []
    for subname in subnames:
        try:
            int_vals.append(int(subname))
# ...
    assert len(int_vals) == 1, (
        f"layer name {layer_name} should" " only contain one integer"
    )
    return int_vals[0]
```
**EN:** This function drives `extract_layer_index` with inputs such as `layer_name`. Extract the layer index from the module name.
**CN:** 这个函数负责 `extract_layer_index`，主要处理 `layer_name` 等输入。 文档字符串说明：Extract the layer index from the module name.

### Lines 89-104: Function `modulate` / 函数 `modulate`
```python
def modulate(
    x: torch.Tensor,
    shift: torch.Tensor | None = None,
    scale: torch.Tensor | None = None,
) -> torch.Tensor:
    """modulate by shift and scale"""
    if scale is None and shift is None:
        return x
    elif shift is None:
        return x * (1 + scale.unsqueeze(1))  # type: ignore[union-attr]
    elif scale is None:
        return x + shift.unsqueeze(1)  # type: ignore[union-attr]
    else:
        return x * (1 + scale.unsqueeze(1)) + shift.unsqueeze(
            1
        )  # type: ignore[union-attr]
```
**EN:** This function drives `modulate` with inputs such as `x`, `shift`, `scale`. modulate by shift and scale
**CN:** 这个函数负责 `modulate`，主要处理 `x`, `shift`, `scale` 等输入。 文档字符串说明：modulate by shift and scale

### Lines 107-152: Function `pred_noise_to_pred_video` / 函数 `pred_noise_to_pred_video`
```python
def pred_noise_to_pred_video(
    pred_noise: torch.Tensor,
    noise_input_latent: torch.Tensor,
    timestep: torch.Tensor,
    scheduler: Any,
) -> torch.Tensor:
    """
    Convert predicted noise to clean latent.

    Args:
    pred_noise: the predicted noise with shape [B, C, H, W]
        where B is batch_size or batch_size * num_frames
    noise_input_latent: the noisy latent with shape [B, C, H, W],
    timestep: the timestep with shape [1] or [bs * num_frames] or [bs, num_frames]
# ...
    )
    sigma_t = sigmas[timestep_id].reshape(-1, 1, 1, 1)
    pred_video = noise_input_latent - sigma_t * pred_noise
    return pred_video.to(dtype)
```
**EN:** This function drives `pred_noise_to_pred_video` with inputs such as `pred_noise`, `noise_input_latent`, `timestep`, `scheduler`. Convert predicted noise to clean latent.
**CN:** 这个函数负责 `pred_noise_to_pred_video`，主要处理 `pred_noise`, `noise_input_latent`, `timestep`, `scheduler` 等输入。 文档字符串说明：Convert predicted noise to clean latent.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Scheduling and batching / 调度与批处理
- Artifact storage management / 产物存储管理
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.srt.utils`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `typing`
