# kimi_common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/kimi_common.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for kimi common models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 kimi common 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4: Documents the scope
```python
"""Kimi-specific grid-based multimodal data helpers.

Shared by KimiVLImageProcessor and KimiK2_5VLImageProcessor.
"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 5-5: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 6-15: Imports dependencies
```python
from typing import Union

import numpy as np
import torch

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
```
**EN:** This block groups related imports for the module, including typing.Union, numpy, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, and 1 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Union, numpy, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem 等 1 项，为后续代码准备所需名称。

### Lines 16-17: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 18-18: Declares class KimiGridMMDataMixin
```python
class KimiGridMMDataMixin:
```
**EN:** This block introduces class `KimiGridMMDataMixin` as a reusable abstraction inside the module. Mixin providing Kimi-specific grid-based multimodal data helpers. Expects the concrete class to supply: - self.hf_config (with vision_config.merge_kernel_size) - self._tokenizer (with .encode())
**CN:** 该代码块声明类 `KimiGridMMDataMixin`，作为模块中的可复用抽象。 文档字符串摘要：Mixin providing Kimi-specific grid-based multimodal data helpers. Expects the concrete class to supply: - self.hf_config (with vision_config.merge_kernel_size) - self._tokenizer (with .encode())

### Lines 19-24: Documents the scope
```python
    """Mixin providing Kimi-specific grid-based multimodal data helpers.

    Expects the concrete class to supply:
      - self.hf_config  (with vision_config.merge_kernel_size)
      - self._tokenizer (with .encode())
    """
```
**EN:** This string literal serves as documentation for the KimiGridMMDataMixin, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 KimiGridMMDataMixin 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 25-25: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiGridMMDataMixin, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiGridMMDataMixin 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-54: Defines function KimiGridMMDataMixin._num_image_tokens_from_grid
```python
    def _num_image_tokens_from_grid(
        self, grid_thw: Union[torch.Tensor, np.ndarray, list, tuple]
    ) -> int:
        """Compute Kimi-style image token count from 2D/3D grid metadata."""
        merge_h, merge_w = self.hf_config.vision_config.merge_kernel_size

        if isinstance(grid_thw, torch.Tensor):
            vals = grid_thw.flatten().tolist()
        elif isinstance(grid_thw, np.ndarray):
            vals = grid_thw.reshape(-1).tolist()
        elif isinstance(grid_thw, (list, tuple)):
            vals = list(np.array(grid_thw).reshape(-1).tolist())
        else:
            raise TypeError(
                f"Unsupported grid type for kimi image tokens: {type(grid_thw)}"
            )

        if len(vals) >= 3:
            _t, h, w = vals[-3], vals[-2], vals[-1]
        elif len(vals) == 2:
            _t, h, w = 1, vals[0], vals[1]
        else:
            raise ValueError(
                f"Invalid grid metadata for kimi image tokens: {vals} "
                "(expected [t,h,w] or [h,w])"
            )

        h, w = int(h), int(w)
        return (h * w) // (merge_h * merge_w)
```
**EN:** This block defines function `KimiGridMMDataMixin._num_image_tokens_from_grid`. Parameters: self, grid_thw. Compute Kimi-style image token count from 2D/3D grid metadata.
**CN:** 该代码块定义函数 `KimiGridMMDataMixin._num_image_tokens_from_grid`。 参数包括 self、grid_thw。 文档字符串摘要：Compute Kimi-style image token count from 2D/3D grid metadata.

### Lines 55-55: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiGridMMDataMixin, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiGridMMDataMixin 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 56-113: Defines function KimiGridMMDataMixin._build_kimi_mm_data_from_grids
```python
    def _build_kimi_mm_data_from_grids(
        self, prompt, embeddings, **kwargs
    ) -> MultimodalProcessorOutput:
        image_token_id = kwargs.get("image_token_id", 0)
        img_grid_thw = kwargs.get("img_grid_thw", None)

        if not isinstance(prompt, list):
            prompt = self._tokenizer.encode(prompt)

        image_token_counts = [
            self._num_image_tokens_from_grid(grid) for grid in img_grid_thw
        ]

        input_ids = []
        offsets = []
        img_idx = 0

        for token in prompt:
            if token != image_token_id:
                input_ids.append(token)
                continue

            if img_idx >= len(image_token_counts):
                raise ValueError(
                    "The number of image placeholders exceeds img_grid_thw entries."
                )

            num_tokens = image_token_counts[img_idx]
            start = len(input_ids)
            input_ids.extend([image_token_id] * num_tokens)
            offsets.append((start, len(input_ids) - 1))
            img_idx += 1

        if img_idx != len(image_token_counts):
            raise ValueError(
                "The number of image placeholders does not match img_grid_thw entries."
            )

        image_embeddings = embeddings[Modality.IMAGE]
        mm_items = []
        consumed = 0
        for start, end in offsets:
            num_tokens = end - start + 1
            embedding_slice = image_embeddings[consumed : consumed + num_tokens]
            consumed += num_tokens
            mm_items.append(
                MultimodalDataItem(
                    modality=Modality.IMAGE,
                    offsets=[(start, end)],
                    precomputed_embeddings=embedding_slice,
                )
            )

        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=mm_items,
            im_token_id=image_token_id,
        )
```
**EN:** This block defines function `KimiGridMMDataMixin._build_kimi_mm_data_from_grids`. Parameters: self, prompt, embeddings.
**CN:** 该代码块定义函数 `KimiGridMMDataMixin._build_kimi_mm_data_from_grids`。 参数包括 self、prompt、embeddings。

## Key Concepts / 关键概念
- **Classes / 类**: `KimiGridMMDataMixin`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `numpy`, `torch`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`
