# evs_core.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/evs/evs_core.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements EVS-related multimodal components for evs core, covering model glue code and runtime integration. / 该模块实现与 evs core 相关的 EVS 多模态组件，涵盖模型衔接代码与运行时集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Comments and module notes
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/multimodal/evs.py

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 18-18: Imports dependencies
```python
import torch
```
**EN:** This block imports dependencies for the module, including torch. These imports supply standard utilities, third-party packages, or SGLang runtime components used later in the file.
**CN:** 该代码块为 模块 导入依赖，包括 torch。这些导入为后续实现提供标准库工具、第三方包或 SGLang 运行时组件。

### Lines 19-20: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 21-40: Defines function compute_retained_tokens_count
```python
def compute_retained_tokens_count(
    tokens_per_frame: int, num_frames: int, q: float
) -> int:
    """
    Compute the number of retained tokens for a given video.
    Method ensures that we retain all the tokens from the first frame
    regardless of the pruning rate.

    Args:
        tokens_per_frame: The number of tokens per frame.
        num_frames: The total number of frames.
        q: The pruning rate.

    Returns:
        The number of retained tokens.
    """
    total_tokens = tokens_per_frame * num_frames
    evs_num_tokens = int(total_tokens * (1 - q))
    min_num_tokens = tokens_per_frame
    return max(min_num_tokens, evs_num_tokens)
```
**EN:** This block defines function `compute_retained_tokens_count`. Parameters: tokens_per_frame, num_frames, q. Compute the number of retained tokens for a given video. Method ensures that we retain all the tokens from the first frame regardless of the pruning rate.
**CN:** 该代码块定义函数 `compute_retained_tokens_count`。 参数包括 tokens_per_frame、num_frames、q。 文档字符串摘要：Compute the number of retained tokens for a given video. Method ensures that we retain all the tokens from the first frame regardless of the pruning rate.

### Lines 41-42: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 43-97: Defines function compute_retention_mask
```python
def compute_retention_mask(
    video_embeds: torch.Tensor,
    video_size_thw: torch.LongTensor | tuple[int, int, int],
    spatial_merge_size: int,
    q: float,
) -> torch.Tensor:
    """
    Computes the retention mask for input video embeddings.

    Args:
        video_embeds (`torch.Tensor`): The input video embeddings
            of shape `(T * H * W // spatial_merge_size ^ 2, hidden_size)`
        video_size_thw (`torch.LongTensor` of shape `(3)`):
            The temporal, height and width of video.
        spatial_merge_size: Size reduction for rows & cols dimensions.
        q: (`float`): Pruning rate factor [0,1)

    Returns:
        `torch.Tensor`: The retention mask for the video embeddings of
            `(T * H * W // spatial_merge_size ^ 2)` shape.
    """
    T, H, W = map(int, video_size_thw)

    # Use reshape instead of einops to avoid graph breaks
    video_embeds = video_embeds.reshape(
        T,
        H // spatial_merge_size,
        W // spatial_merge_size,
        video_embeds.size(-1),
    )
    tokens_per_frame = (H // spatial_merge_size) * (W // spatial_merge_size)
    # Core EVS
    similarity = torch.nn.functional.cosine_similarity(
        video_embeds[1:, ...], video_embeds[:-1, ...], dim=-1
    )
    dissimilarity = 1 - similarity

    # Always ensure we include all tokens from the first frame
    dissimilarity = torch.cat(
        [255 * torch.ones_like(video_embeds[:1, :, :, 0]), dissimilarity], dim=0
    )

    dissimilarity_flat = dissimilarity.view(-1)
    order = torch.argsort(dissimilarity_flat, dim=-1, descending=True, stable=True)
    retain_num_tokens = compute_retained_tokens_count(
        tokens_per_frame=tokens_per_frame, num_frames=T, q=q
    )
    topk_indices = order[:retain_num_tokens]

    retention_mask = torch.zeros_like(dissimilarity_flat, dtype=torch.bool)
    retention_mask[topk_indices] = True
    retention_mask = retention_mask.reshape(dissimilarity.size())

    mask = retention_mask.view(-1)  # "T H W -> (T H W)"
    return mask
```
**EN:** This block defines function `compute_retention_mask`. Parameters: video_embeds, video_size_thw, spatial_merge_size, q. Computes the retention mask for input video embeddings. Args: video_embeds (`torch.Tensor`): The input video embeddings of shape `(T * H * W // spatial_merge_size ^ 2, hidden_size)` video_size_thw (`torch.LongTensor` of shape `(3)`): The temporal, height and w
**CN:** 该代码块定义函数 `compute_retention_mask`。 参数包括 video_embeds、video_size_thw、spatial_merge_size、q。 文档字符串摘要：Computes the retention mask for input video embeddings. Args: video_embeds (`torch.Tensor`): The input video embeddings of shape `(T * H * W // spatial_merge_size ^ 2, hidden_size)` video_size_thw (`torch.LongTensor` of shape `(3)`): The temporal, height and w

### Lines 98-102: Comments and module notes
```python


# ▲ End of VLLM code


```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 103-121: Defines function tokens_per_frame
```python
def tokens_per_frame(
    *,
    q: float,
    num_frames: int,
    frame_num_tokens: int,
) -> list[int]:
    """
    Before EVS pruning, we want to pre-reduce input_ids to be the same length that will be retained of embeddings due to EVS pruning, so the forward batch metadata will be correct post EVS.
    We don't know the exact number of tokens per frame after EVS pruning, but we know the *total* number of tokens that will be retained.
    So, we create a bogus tokens_per_frame list that sums to the total number of tokens that will be retained, and use it for placeholder spans, later to replaced, see `replace_offsets_with_tokens_per_frame` below.
    """
    retained = compute_retained_tokens_count(
        tokens_per_frame=frame_num_tokens, num_frames=num_frames, q=q
    )
    base = retained // num_frames
    rem = retained % num_frames
    tpf = [base] * (num_frames - 1) + [base + rem]
    assert sum(tpf) == retained
    return tpf
```
**EN:** This block defines function `tokens_per_frame`. It takes no explicit parameters. Before EVS pruning, we want to pre-reduce input_ids to be the same length that will be retained of embeddings due to EVS pruning, so the forward batch metadata will be correct post EVS. We don't know the exact number of tokens per frame after EVS pruning, but
**CN:** 该代码块定义函数 `tokens_per_frame`。 它没有显式参数。 文档字符串摘要：Before EVS pruning, we want to pre-reduce input_ids to be the same length that will be retained of embeddings due to EVS pruning, so the forward batch metadata will be correct post EVS. We don't know the exact number of tokens per frame after EVS pruning, but

### Lines 122-123: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 124-178: Defines function replace_offsets_with_tokens_per_frame
```python
def replace_offsets_with_tokens_per_frame(
    *,
    pre_chunked_input_ids: list[int],
    num_tokens_per_frame: list[int],
    frame_offsets_inclusive: list[tuple[int, int]],
    filler_token_id: int,
) -> list[int]:
    """
    Given a single video, after EVS pruning of redundant tokens, we have a new `num_tokens_per_frame`, therefore the existing input_ids and offsets are stale.
    We need to replace all stale offsets with new offsets that reflect the new `num_tokens_per_frame`, respectively.

    Returns:
        Modified input_ids with offsets replaced with new offsets.

    Examples:
    >>> assert replace_offsets_with_tokens_per_frame(
    ...     pre_chunked_input_ids=[1, 0, 0, 4, 5, 0, 0, 0, 9, 10, 0, 0, 12, 13],
    ...     frame_offsets_inclusive=[(1, 2), (5, 7), (10, 11)],
    ...     num_tokens_per_frame=[1, 4, 2],
    ...     filler_token_id=0,
    ... ) ==                      [1, 0, 4, 5, 0, 0, 0, 0, 9, 10, 0, 0, 12, 13]

    >>> assert replace_offsets_with_tokens_per_frame(
    ...     pre_chunked_input_ids=[1, 0, 0, 4, 5, 9, 10, 0, 0, 0],
    ...     frame_offsets_inclusive=[(1, 2), (7, 9)],
    ...     num_tokens_per_frame=[1, 4],
    ...     filler_token_id=0,
    ... ) ==                      [1, 0, 4, 5, 9, 10, 0, 0, 0, 0]

    >>> assert replace_offsets_with_tokens_per_frame(
    ...     pre_chunked_input_ids=[0, 0, 1, 4, 0, 0, 0, 5, 9, 10],
    ...     frame_offsets_inclusive=[(0, 1), (4, 6)],
    ...     num_tokens_per_frame=[1, 4],
    ...     filler_token_id=0,
    ... ) ==                      [0, 1, 4, 0, 0, 0, 0, 5, 9, 10]
    """
    assert isinstance(pre_chunked_input_ids, list)
    ids = pre_chunked_input_ids

    if len(frame_offsets_inclusive) == 1:
        """There might be no frame separators, in which case there will be one contiguous span of tokens"""
        final = ids[0 : frame_offsets_inclusive[0][0]]
        frames = [filler_token_id] * sum(num_tokens_per_frame)
        final.extend(frames)
    else:
        cursor = 0
        final = []
        for (start, end), num_tokens in zip(
            frame_offsets_inclusive, num_tokens_per_frame, strict=True
        ):
            final.extend(ids[cursor:start])
            final.extend([filler_token_id] * num_tokens)
            cursor = end + 1
    final.extend(ids[frame_offsets_inclusive[-1][1] + 1 :])
    return final
```
**EN:** This block defines function `replace_offsets_with_tokens_per_frame`. It takes no explicit parameters. Given a single video, after EVS pruning of redundant tokens, we have a new `num_tokens_per_frame`, therefore the existing input_ids and offsets are stale. We need to replace all stale offsets with new offsets that reflect the new `num_tokens_per_frame`, respec
**CN:** 该代码块定义函数 `replace_offsets_with_tokens_per_frame`。 它没有显式参数。 文档字符串摘要：Given a single video, after EVS pruning of redundant tokens, we have a new `num_tokens_per_frame`, therefore the existing input_ids and offsets are stale. We need to replace all stale offsets with new offsets that reflect the new `num_tokens_per_frame`, respec

## Key Concepts / 关键概念
- **Functions / 函数**: `compute_retained_tokens_count`, `compute_retention_mask`, `tokens_per_frame`, `replace_offsets_with_tokens_per_frame`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `torch`
