# evs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/evs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements evs support for the `multimodal` portion of vLLM. / 为 vLLM 的 `multimodal` 子目录实现与 evs 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 11-13)
```python
import typing

import torch
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Function `compute_retained_tokens_count` (lines 16-35)
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
**EN:** Function `compute_retained_tokens_count` implements an encoding/decoding or token-transformation step. The docstring highlights: Compute the number of retained tokens for a given video. Key calls such as `int`, `max` show the concrete execution path.
**CN:** Function `compute_retained_tokens_count` 实现编码/解码或 Token 变换步骤。 文档字符串强调：Compute the number of retained tokens for a given video. 像 `int`, `max` 这样的关键调用展示了该代码块的具体执行路径。

### Function `compute_retention_mask` (lines 38-92)
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
    # ... omitted for brevity ...
    mask = retention_mask.view(-1)  # "T H W -> (T H W)"
    return mask
```
**EN:** Function `compute_retention_mask` provides a reusable helper around the module's main workflow. The docstring highlights: Computes the retention mask for input video embeddings. Key calls such as `map`, `video_embeds.reshape`, `video_embeds.size`, `torch.nn.functional.cosine_similarity`, `torch.cat` show the concrete execution path.
**CN:** Function `compute_retention_mask` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Computes the retention mask for input video embeddings. 像 `map`, `video_embeds.reshape`, `video_embeds.size`, `torch.nn.functional.cosine_similarity`, `torch.cat` 这样的关键调用展示了该代码块的具体执行路径。

### Function `compute_mrope_for_media` (lines 95-151)
```python
def compute_mrope_for_media(
    video_size_thw: torch.LongTensor,
    spatial_merge_size: int,
    tokens_per_second: float = 1.0,
    video_second_per_grid: float = 1.0,
) -> torch.Tensor:
    """
    Computes the mrope for video embeddings based on the grid dimensions.
    Computed mrope positions match original qwen 2.5 implementation,
    but positions are built for media being the first element in sequence.

    Args:
        video_size_thw: Media size (num frames, rows, cols)
        spatial_merge_size: Size reduction for rows & cols dimensions.
        tokens_per_second: Number of tokens per second.
        video_second_per_grid: Number of seconds per video.

    Returns:
        Tensor of shape `(T * H * W, 4)` where last dimension
        represents mrope positions [0:3), while the last channel
        contains value of llm_grid_w repeated for all positions.
    """
    llm_grid_t = video_size_thw[0]
    llm_grid_h = video_size_thw[1] // spatial_merge_size
    llm_grid_w = video_size_thw[2] // spatial_merge_size
    # ... omitted for brevity ...
    positions = torch.stack([t_index, h_index, w_index, llm_grid_w], dim=1)
    return positions
```
**EN:** Function `compute_mrope_for_media` provides a reusable helper around the module's main workflow. The docstring highlights: Computes the mrope for video embeddings based on the grid dimensions. Key calls such as `torch.arange(llm_grid_t).view(-1, 1).expand(-1, llm_grid_h * llm_grid_w).mul(tokens_per_second * video_second_per_grid).long().flatten`, `torch.arange(llm_grid_t).view(-1, 1).expand(-1, llm_grid_h * llm_grid_w).mul(tokens_per_second * video_second_per_grid).long`, `torch.arange(llm_grid_t).view(-1, 1).expand(-1, llm_grid_h * llm_grid_w).mul`, `torch.arange(llm_grid_t).view(-1, 1).expand`, `torch.arange(llm_grid_t).view` show the concrete execution path.
**CN:** Function `compute_mrope_for_media` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Computes the mrope for video embeddings based on the grid dimensions. 像 `torch.arange(llm_grid_t).view(-1, 1).expand(-1, llm_grid_h * llm_grid_w).mul(tokens_per_second * video_second_per_grid).long().flatten`, `torch.arange(llm_grid_t).view(-1, 1).expand(-1, llm_grid_h * llm_grid_w).mul(tokens_per_second * video_second_per_grid).long`, `torch.arange(llm_grid_t).view(-1, 1).expand(-1, llm_grid_h * llm_grid_w).mul`, `torch.arange(llm_grid_t).view(-1, 1).expand`, `torch.arange(llm_grid_t).view` 这样的关键调用展示了该代码块的具体执行路径。

### Function `recompute_mrope_positions` (lines 154-356)
```python
def recompute_mrope_positions(
    input_ids: torch.LongTensor,
    multimodal_positions: list[torch.Tensor],
    mrope_positions: torch.LongTensor,
    num_computed_tokens: int,
    vision_start_token_id: int,
    image_token_id: int,
    video_token_id: int,
) -> tuple[torch.LongTensor, int]:
    """
    Update part of input mrope positions.
    Original mrope_positions are computed incorrectly, so once we prune media
    tokens we should reflect this in the mrope positions for the LLM.

    This method supports chunked prefill approach where
    multimodal_embeddings are passed to LLM in chunks, so input
    multimodal_embeddings may contain zero, some or even some part of all
    multimodal_embeddings for a given prompt.

    Each multimodal_positions has 4 or 5 extra channels
    (first 3 channels correspond to the original 3 mrope positions;
    remaining channels vary by model — see below). Provided multimodal_positions
    do not reflect location of media position in sequence - they are computed
    like the media is in the 0-th position in the sequence.

    # ... omitted for brevity ...
    mrope_positions_delta = (positions.max() + 1 - N).item()
    return positions, mrope_positions_delta
```
**EN:** Function `recompute_mrope_positions` provides a reusable helper around the module's main workflow. The docstring highlights: Update part of input mrope positions. Key calls such as `typing.cast`, `mrope_positions.clone`, `input_ids.numel`, `input_ids.eq`, `len` show the concrete execution path.
**CN:** Function `recompute_mrope_positions` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Update part of input mrope positions. 像 `typing.cast`, `mrope_positions.clone`, `input_ids.numel`, `input_ids.eq`, `len` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import typing`
- **Third-party / 第三方**: `import torch`
