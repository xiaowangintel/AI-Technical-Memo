# mrope_rope_index.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/mrope_rope_index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements rotary position embedding logic with embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `_get_feat_extract_output_lengths`, `_get_llm_pos_ids_for_vision`, `get_rope_index`, and `get_rope_index_qwen3_omni` and connects them to backend-specific paths such as `NPU` and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了旋转位置编码逻辑，并结合嵌入查找与预处理。它提供了 `_get_feat_extract_output_lengths`、`_get_llm_pos_ids_for_vision`、`get_rope_index` 以及 `get_rope_index_qwen3_omni` 等符号，并把这些符号连接到 `NPU` 和 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module imports and dependency wiring
```python
"""get_rope_index implementations for Qwen2-VL/Qwen3-VL, Qwen3-Omni, GLM4V, Ernie4.5."""

from __future__ import annotations

import itertools
from typing import Any, List, Optional, Tuple, Union

import torch
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `itertools`, `typing.Any`, `typing.List`, `typing.Optional`, and `typing.Tuple`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`itertools`、`typing.Any`、`typing.List`、`typing.Optional` 以及 `typing.Tuple`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 11-22: Internal helper `_get_feat_extract_output_lengths`
```python
def _get_feat_extract_output_lengths(input_lengths):
    """
    Computes the output length of the convolutional layers and the output length of the audio encoder
    """
    input_lengths_leave = input_lengths % 100
    feat_lengths = (input_lengths_leave - 1) // 2 + 1
    output_lengths = (
        ((feat_lengths - 1) // 2 + 1 - 1) // 2 + 1 + (input_lengths // 100) * 13
    )
    return output_lengths
```
**EN:** This block defines `_get_feat_extract_output_lengths` and contains the main logic for this step. Intermediate names such as `input_lengths_leave`, `feat_lengths`, and `output_lengths` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_get_feat_extract_output_lengths`，并承载这一阶段的核心逻辑。 像 `input_lengths_leave`、`feat_lengths` 以及 `output_lengths` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 23-46: Internal helper `_get_llm_pos_ids_for_vision`
```python
def _get_llm_pos_ids_for_vision(
    st_idx, vision_idx, spatial_merge_size, t_index, grid_hs, grid_ws, device
):
    grid_h = grid_hs[vision_idx] // spatial_merge_size
    grid_w = grid_ws[vision_idx] // spatial_merge_size

    h_index = (
        torch.arange(grid_h, device=device)
        .view(1, -1, 1)
        .expand(len(t_index), -1, grid_w)
        .flatten()
    )
    w_index = (
        torch.arange(grid_w, device=device)
        .view(1, 1, -1)
        .expand(len(t_index), grid_h, -1)
        .flatten()
    )
    t_index = t_index.view(-1, 1).expand(-1, grid_h * grid_w).flatten()

    llm_pos_ids = torch.stack([t_index, h_index, w_index], dim=0) + st_idx
    return llm_pos_ids
```
**EN:** This block defines `_get_llm_pos_ids_for_vision` and contains the main logic for this step. It mainly invokes `torch.arange.view.expand.flatten`, `t_index.view.expand.flatten`, `torch.stack`, `torch.arange.view.expand`, and `t_index.view.expand`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `grid_h`, `grid_w`, `h_index`, `w_index`, and `t_index` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_get_llm_pos_ids_for_vision`，并承载这一阶段的核心逻辑。 它主要调用 `torch.arange.view.expand.flatten`、`t_index.view.expand.flatten`、`torch.stack`、`torch.arange.view.expand` 以及 `t_index.view.expand`，说明该流程会编排底层辅助函数或计算内核。 像 `grid_h`、`grid_w`、`h_index`、`w_index` 以及 `t_index` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 47-213: `get_rope_index` getter for RoPE index
```python
def get_rope_index(
    spatial_merge_size: int,
    image_token_id: int,
    video_token_id: int,
    vision_start_token_id: int,
    model_type: str,
    tokens_per_second: Optional[int] = None,
    input_ids: Optional[torch.LongTensor] = None,
    image_grid_thw: Optional[torch.LongTensor] = None,
    video_grid_thw: Optional[torch.LongTensor] = None,
    second_per_grid_ts: Optional[torch.Tensor] = None,
    **kwargs,
) -> Tuple[torch.Tensor, torch.Tensor]:
    if model_type == "qwen3_omni_moe":
        return get_rope_index_qwen3_omni(
            spatial_merge_size,
            image_token_id,
            video_token_id,
            vision_start_token_id,
            tokens_per_second,
            input_ids,
            image_grid_thw,
            video_grid_thw,
            second_per_grid_ts,
            **kwargs,
        )
    if (
        model_type.startswith("qwen3_vl")
        or model_type.startswith("qwen3_vl_moe")
        or model_type.startswith("qwen3_5")
    ) and video_grid_thw is not None:
        video_grid_thw = torch.repeat_interleave(
            video_grid_thw, video_grid_thw[:, 0], dim=0
        )
        video_grid_thw[:, 0] = 1

    mrope_position_deltas = []
    if input_ids is not None and (
        image_grid_thw is not None or video_grid_thw is not None
    ):
        total_input_ids = input_ids
        position_ids = torch.ones(
            3,
            input_ids.shape[0],
            input_ids.shape[1],
            dtype=input_ids.dtype,
            device=input_ids.device,
        )
        image_index, video_index = 0, 0
        for i, input_ids in enumerate(total_input_ids):
            image_nums, video_nums = 0, 0
            vision_start_indices = torch.argwhere(
                input_ids == vision_start_token_id
            ).squeeze(1)
            vision_tokens = input_ids[vision_start_indices + 1]
            image_nums = (vision_tokens == image_token_id).sum()
            video_nums = (vision_tokens == video_token_id).sum()
            input_tokens = input_ids.tolist()
            llm_pos_ids_list: list = []
            st = 0
            remain_images, remain_videos = image_nums, video_nums
            for _ in range(image_nums + video_nums):
                if image_token_id in input_tokens and remain_images > 0:
                    ed_image = input_tokens.index(image_token_id, st)
                else:
                    ed_image = len(input_tokens) + 1
                if video_token_id in input_tokens and remain_videos > 0:
                    ed_video = input_tokens.index(video_token_id, st)
                else:
                    ed_video = len(input_tokens) + 1
                if ed_image < ed_video:
                    t, h, w = (
                        image_grid_thw[image_index][0],
                        image_grid_thw[image_index][1],
                        image_grid_thw[image_index][2],
                    )
                    second_per_grid_t = 0
                    image_index += 1
                    remain_images -= 1
                    ed = ed_image
                else:
                    t, h, w = (
                        video_grid_thw[video_index][0],
                        video_grid_thw[video_index][1],
                        video_grid_thw[video_index][2],
                    )
                    if second_per_grid_ts is not None:
                        second_per_grid_t = second_per_grid_ts[video_index]
                    else:
                        second_per_grid_t = 1.0
                    video_index += 1
                    remain_videos -= 1
                    ed = ed_video
                t_int, h_int, w_int = int(t), int(h), int(w)
                llm_grid_t = t_int
                llm_grid_h = h_int // spatial_merge_size
                llm_grid_w = w_int // spatial_merge_size
                text_len = ed - st
                st_idx = (
                    llm_pos_ids_list[-1].max() + 1 if len(llm_pos_ids_list) > 0 else 0
                )
                llm_pos_ids_list.append(
                    torch.arange(text_len).view(1, -1).expand(3, -1) + st_idx
                )
                if model_type in ("qwen2_5_vl", "paddleocr_vl"):
                    range_tensor = torch.arange(llm_grid_t).view(-1, 1)
                    expanded_range = range_tensor.expand(-1, llm_grid_h * llm_grid_w)
                    time_tensor = expanded_range * second_per_grid_t * tokens_per_second
                    t_index = time_tensor.long().flatten()
                elif model_type in (
                    "qwen2_vl",
                    "qwen3_vl",
                    "qwen3_vl_moe",
                    "qwen3_5",
                    "qwen3_5_moe",
                    "intern_s2_preview",
                ):
                    t_index = (
                        torch.arange(llm_grid_t, device=position_ids.device)
                        .view(-1, 1)
                        .expand(llm_grid_t, llm_grid_h * llm_grid_w)
                        .reshape(-1)
                    )
                else:
                    raise RuntimeError(f"Unimplemented model type: {model_type}")
                h_index = (
                    torch.arange(llm_grid_h, device=position_ids.device)
                    .view(1, -1, 1)
                    .expand(llm_grid_t, llm_grid_h, llm_grid_w)
                    .reshape(-1)
                )
                w_index = (
                    torch.arange(llm_grid_w, device=position_ids.device)
                    .view(1, 1, -1)
                    .expand(llm_grid_t, llm_grid_h, llm_grid_w)
                    .reshape(-1)
                )
                llm_pos_ids_list.append(
                    torch.stack([t_index, h_index, w_index]) + text_len + st_idx
                )
                st = ed + llm_grid_t * llm_grid_h * llm_grid_w
            if st < len(input_tokens):
                st_idx = (
                    llm_pos_ids_list[-1].max() + 1 if len(llm_pos_ids_list) > 0 else 0
                )
                text_len = len(input_tokens) - st
                llm_pos_ids_list.append(
                    torch.arange(text_len).view(1, -1).expand(3, -1) + st_idx
                )
            llm_positions = torch.cat(llm_pos_ids_list, dim=1).reshape(3, -1)
            position_ids[..., i, :] = llm_positions.to(position_ids.device)
            mrope_position_deltas.append(
                llm_positions.max() + 1 - len(total_input_ids[i])
            )
        mrope_position_deltas = torch.tensor(
            mrope_position_deltas, device=input_ids.device
        ).unsqueeze(1)
        return position_ids, mrope_position_deltas
    else:
        s = input_ids.shape[1]
        position_ids = torch.arange(s)
        position_ids = position_ids.unsqueeze(0).expand(3, -1, -1).to(input_ids.device)
        max_position_ids = position_ids.amax(dim=0, keepdim=False)
        mrope_position_deltas = max_position_ids.amax(-1, keepdim=True) + 1 - s
        return position_ids, mrope_position_deltas
```
**EN:** This block defines `get_rope_index` and contains the main logic for this step. It mainly invokes `get_rope_index_qwen3_omni`, `torch.repeat_interleave`, `torch.ones`, `enumerate`, and `torch.tensor.unsqueeze`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `mrope_position_deltas`, `video_grid_thw`, `total_input_ids`, `position_ids`, and `image_index` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_rope_index`，并承载这一阶段的核心逻辑。 它主要调用 `get_rope_index_qwen3_omni`、`torch.repeat_interleave`、`torch.ones`、`enumerate` 以及 `torch.tensor.unsqueeze`，说明该流程会编排底层辅助函数或计算内核。 像 `mrope_position_deltas`、`video_grid_thw`、`total_input_ids`、`position_ids` 以及 `image_index` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 214-480: `get_rope_index_qwen3_omni` getter for RoPE index qwen 3 omni
```python
def get_rope_index_qwen3_omni(
    spatial_merge_size: int,
    image_token_id: int,
    video_token_id: int,
    vision_start_token_id: int,
    tokens_per_second: Optional[int] = None,
    input_ids: Optional[torch.LongTensor] = None,
    image_grid_thw: Optional[torch.LongTensor] = None,
    video_grid_thw: Optional[torch.LongTensor] = None,
    second_per_grid_ts: Optional[torch.Tensor] = None,
    **kwargs,
) -> Tuple[torch.Tensor, torch.Tensor]:
    audio_token_id = kwargs["audio_token_id"]
    audio_start_token_id = kwargs["audio_start_token_id"]
    position_id_per_seconds = kwargs["position_id_per_seconds"]
    use_audio_in_video = kwargs.get("use_audio_in_video", False)
    audio_seqlens = kwargs.get("audio_seqlens", None)
    second_per_grids = second_per_grid_ts

    mrope_position_deltas = []
    if input_ids is not None and (
        image_grid_thw is not None or video_grid_thw is not None
    ):
        total_input_ids = input_ids
        position_ids = torch.zeros(
            3,
            input_ids.shape[0],
            input_ids.shape[1],
            dtype=torch.float,
            device=input_ids.device,
        )
        image_idx, video_idx, audio_idx = 0, 0, 0
        for i, current_input_ids in enumerate(total_input_ids):
            image_nums, video_nums, audio_nums = 0, 0, 0
            vision_start_indices = torch.argwhere(
                current_input_ids == vision_start_token_id
            ).squeeze(1)
            if vision_start_indices.numel() > 0:
                vision_tokens = current_input_ids[vision_start_indices + 1]
                image_nums = (vision_tokens == image_token_id).sum()
                video_nums = (
                    (vision_tokens == audio_start_token_id).sum()
                    if use_audio_in_video
                    else (vision_tokens == video_token_id).sum()
                )
            audio_nums = torch.sum(current_input_ids == audio_start_token_id)
            input_tokens = current_input_ids.tolist()
            llm_pos_ids_list: list = []
            st = 0
            remain_images, remain_videos, remain_audios = (
                image_nums,
                video_nums,
                audio_nums,
            )
            multimodal_nums = (
                image_nums + audio_nums
                if use_audio_in_video
                else image_nums + video_nums + audio_nums
            )
            for _ in range(multimodal_nums):
                st_idx = (
                    llm_pos_ids_list[-1].max() + 1 if len(llm_pos_ids_list) > 0 else 0
                )
                ed_vision_start = (
                    input_tokens.index(vision_start_token_id, st)
                    if (
                        (
                            image_token_id in input_tokens
                            or video_token_id in input_tokens
                        )
                        and (remain_videos > 0 or remain_images > 0)
                    )
                    else len(input_tokens) + 1
                )
                ed_audio_start = (
                    input_tokens.index(audio_start_token_id, st)
                    if (audio_token_id in input_tokens and remain_audios > 0)
                    else len(input_tokens) + 1
                )
                min_ed = min(ed_vision_start, ed_audio_start)
                text_len = min_ed - st
                if text_len != 0:
                    llm_pos_ids_list.append(
                        torch.arange(text_len).view(1, -1).expand(3, -1) + st_idx
                    )
                    st_idx += text_len
                if min_ed == ed_vision_start and ed_vision_start + 1 == ed_audio_start:
                    bos_len, eos_len = 2, 2
                else:
                    bos_len, eos_len = 1, 1
                llm_pos_ids_list.append(
                    torch.arange(bos_len).view(1, -1).expand(3, -1) + st_idx
                )
                st_idx += bos_len
                # Audio Only
                if min_ed == ed_audio_start:
                    audio_len = _get_feat_extract_output_lengths(
                        audio_seqlens[audio_idx]
                    )
                    llm_pos_ids = (
                        torch.arange(audio_len).view(1, -1).expand(3, -1) + st_idx
                    )
                    llm_pos_ids_list.append(llm_pos_ids)
                    st += int(text_len + bos_len + audio_len + eos_len)
                    audio_idx += 1
                    remain_audios -= 1
                # Image Only
                elif (
                    min_ed == ed_vision_start
                    and current_input_ids[ed_vision_start + 1] == image_token_id
                ):
                    grid_t = image_grid_thw[image_idx][0]
                    grid_hs = image_grid_thw[:, 1]
                    grid_ws = image_grid_thw[:, 2]
                    t_index = (
                        torch.arange(grid_t) * 1 * position_id_per_seconds
                    ).float()
                    llm_pos_ids = _get_llm_pos_ids_for_vision(
                        st_idx,
                        image_idx,
                        spatial_merge_size,
                        t_index,
                        grid_hs,
                        grid_ws,
                        input_ids.device,
                    )
                    image_len = image_grid_thw[image_idx].prod() // (
                        spatial_merge_size**2
                    )
                    llm_pos_ids_list.append(llm_pos_ids)
                    st += int(text_len + bos_len + image_len + eos_len)
                    image_idx += 1
                    remain_images -= 1
                # Video Only
                elif (
                    min_ed == ed_vision_start
                    and current_input_ids[ed_vision_start + 1] == video_token_id
                ):
                    grid_t = video_grid_thw[video_idx][0]
                    grid_hs = video_grid_thw[:, 1]
                    grid_ws = video_grid_thw[:, 2]
                    t_index = (
                        torch.arange(grid_t)
                        * second_per_grids[video_idx].cpu().float()
                        * position_id_per_seconds
                    ).float()
                    llm_pos_ids = _get_llm_pos_ids_for_vision(
                        st_idx,
                        video_idx,
                        spatial_merge_size,
                        t_index,
                        grid_hs,
                        grid_ws,
                        input_ids.device,
                    )
                    video_len = video_grid_thw[video_idx].prod() // (
                        spatial_merge_size**2
                    )
                    llm_pos_ids_list.append(llm_pos_ids)
                    st += int(text_len + bos_len + video_len + eos_len)
                    video_idx += 1
                    remain_videos -= 1
                # Audio in Video
                elif (
                    min_ed == ed_vision_start and ed_vision_start + 1 == ed_audio_start
                ):
                    audio_len = _get_feat_extract_output_lengths(
                        audio_seqlens[audio_idx]
                    )
                    audio_llm_pos_ids = (
                        torch.arange(audio_len).view(1, -1).expand(3, -1) + st_idx
                    )
                    grid_t = video_grid_thw[video_idx][0]
                    grid_hs = video_grid_thw[:, 1]
                    grid_ws = video_grid_thw[:, 2]
                    t_index = (
                        torch.arange(grid_t)
                        * second_per_grids[video_idx].cpu().float()
                        * position_id_per_seconds
                    ).float()
                    video_llm_pos_ids = _get_llm_pos_ids_for_vision(
                        st_idx,
                        video_idx,
                        spatial_merge_size,
                        t_index,
                        grid_hs,
                        grid_ws,
                        input_ids.device,
                    )
                    video_data_index, audio_data_index = 0, 0
                    while (
                        video_data_index < video_llm_pos_ids.shape[-1]
                        and audio_data_index < audio_llm_pos_ids.shape[-1]
                    ):
                        if (
                            video_llm_pos_ids[0][video_data_index]
                            <= audio_llm_pos_ids[0][audio_data_index]
                        ):
                            llm_pos_ids_list.append(
                                video_llm_pos_ids[
                                    :, video_data_index : video_data_index + 1
                                ]
                            )
                            video_data_index += 1
                        else:
                            llm_pos_ids_list.append(
                                audio_llm_pos_ids[
                                    :, audio_data_index : audio_data_index + 1
                                ]
                            )
                            audio_data_index += 1
                    if video_data_index < video_llm_pos_ids.shape[-1]:
                        llm_pos_ids_list.append(
                            video_llm_pos_ids[
                                :, video_data_index : video_llm_pos_ids.shape[-1]
                            ]
                        )
                    if audio_data_index < audio_llm_pos_ids.shape[-1]:
                        llm_pos_ids_list.append(
                            audio_llm_pos_ids[
                                :, audio_data_index : audio_llm_pos_ids.shape[-1]
                            ]
                        )
                    video_len = video_grid_thw[video_idx].prod() // (
                        spatial_merge_size**2
                    )
                    st += int(text_len + bos_len + audio_len + video_len + eos_len)
                    audio_idx += 1
                    video_idx += 1
                    remain_videos -= 1
                    remain_audios -= 1
                st_idx = (
                    llm_pos_ids_list[-1].max() + 1 if len(llm_pos_ids_list) > 0 else 0
                )
                llm_pos_ids_list.append(
                    torch.arange(eos_len).view(1, -1).expand(3, -1) + st_idx
                )
            if st < len(input_tokens):
                st_idx = (
                    llm_pos_ids_list[-1].max() + 1 if len(llm_pos_ids_list) > 0 else 0
                )
                text_len = len(input_tokens) - st
                llm_pos_ids_list.append(
                    torch.arange(text_len).view(1, -1).expand(3, -1) + st_idx
                )
            llm_positions = torch.cat(
                [item.float() for item in llm_pos_ids_list], dim=1
            ).reshape(3, -1)
            position_ids[..., i, :] = llm_positions.to(position_ids.device)
            mrope_position_deltas.append(
                llm_positions.max() + 1 - len(current_input_ids)
            )
        mrope_position_deltas = torch.tensor(
            mrope_position_deltas, device=input_ids.device
        ).unsqueeze(1)
        return position_ids, mrope_position_deltas
    else:
        s = input_ids.shape[1]
        position_ids = torch.arange(s)
        position_ids = position_ids.unsqueeze(0).expand(3, -1, -1).to(input_ids.device)
        max_position_ids = position_ids.max(0, keepdim=False)[0].max(-1, keepdim=True)[
            0
        ]
        mrope_position_deltas = max_position_ids + 1 - s
        return position_ids, mrope_position_deltas
```
**EN:** This block defines `get_rope_index_qwen3_omni` and contains the main logic for this step. It mainly invokes `kwargs.get`, `torch.zeros`, `enumerate`, `torch.tensor.unsqueeze`, and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `audio_token_id`, `audio_start_token_id`, `position_id_per_seconds`, `use_audio_in_video`, and `audio_seqlens` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_rope_index_qwen3_omni`，并承载这一阶段的核心逻辑。 它主要调用 `kwargs.get`、`torch.zeros`、`enumerate`、`torch.tensor.unsqueeze` 以及 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `audio_token_id`、`audio_start_token_id`、`position_id_per_seconds`、`use_audio_in_video` 以及 `audio_seqlens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 481-668: `get_rope_index_glm4v` getter for RoPE index glm 4 v
```python
def get_rope_index_glm4v(
    input_ids: torch.Tensor,
    hf_config: Any,
    image_grid_thw: Union[List[List[int]], torch.Tensor],
    video_grid_thw: Union[List[List[int]], torch.Tensor],
    attention_mask: torch.Tensor,
    **kwargs,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Get mrope input positions and delta value for GLM4V."""
    image_token_id = hf_config.image_token_id
    video_start_token_id = hf_config.video_start_token_id
    video_end_token_id = hf_config.video_end_token_id
    spatial_merge_size = hf_config.vision_config.spatial_merge_size

    mrope_position_deltas = []

    if input_ids is not None and (
        image_grid_thw is not None or video_grid_thw is not None
    ):
        total_input_ids = input_ids

        if attention_mask is None:
            attention_mask = torch.ones_like(total_input_ids)

        position_ids = torch.ones(
            3,
            input_ids.shape[0],
            input_ids.shape[1],
            dtype=input_ids.dtype,
            device=input_ids.device,
        )

        image_index, video_index = 0, 0
        video_group_index = 0
        attention_mask = attention_mask.to(total_input_ids.device)

        for i, ids in enumerate(total_input_ids):
            curr_mask = attention_mask[i]
            ids_masked = ids[curr_mask == 1]

            input_tokens = ids_masked.tolist()
            input_token_type = [""] * len(input_tokens)

            video_check_flg = False
            for j, token in enumerate(input_tokens):
                if token == video_start_token_id:
                    video_check_flg = True
                elif token == video_end_token_id:
                    video_check_flg = False

                if token == image_token_id and not video_check_flg:
                    input_token_type[j] = "image"
                elif token == image_token_id and video_check_flg:
                    input_token_type[j] = "video"
                else:
                    input_token_type[j] = "text"

            input_type_group = []
            for key, group in itertools.groupby(
                enumerate(input_token_type), lambda x: x[1]
            ):
                group = list(group)
                start_index = group[0][0]
                end_index = group[-1][0] + 1
                input_type_group.append((key, start_index, end_index))

            llm_pos_ids_list = []
            video_frame_num = 1

            for modality_type, start_idx, end_idx in input_type_group:
                if llm_pos_ids_list:
                    st_idx = llm_pos_ids_list[-1].max().item() + 1
                else:
                    st_idx = 0

                if modality_type == "image":
                    t, h, w = (
                        image_grid_thw[image_index][0],
                        image_grid_thw[image_index][1],
                        image_grid_thw[image_index][2],
                    )
                    t_int, h_int, w_int = int(t), int(h), int(w)
                    llm_grid_t = t_int
                    llm_grid_h = h_int // spatial_merge_size
                    llm_grid_w = w_int // spatial_merge_size

                    t_index = (
                        torch.arange(llm_grid_t, device=position_ids.device)
                        .view(-1, 1)
                        .expand(llm_grid_t, llm_grid_h * llm_grid_w)
                        .reshape(-1)
                    )
                    h_index = (
                        torch.arange(llm_grid_h, device=position_ids.device)
                        .view(1, -1, 1)
                        .expand(llm_grid_t, llm_grid_h, llm_grid_w)
                        .reshape(-1)
                    )
                    w_index = (
                        torch.arange(llm_grid_w, device=position_ids.device)
                        .view(1, 1, -1)
                        .expand(llm_grid_t, llm_grid_h, llm_grid_w)
                        .reshape(-1)
                    )
                    llm_pos_ids_list.append(
                        torch.stack([t_index, h_index, w_index]) + st_idx
                    )
                    image_index += 1
                    video_frame_num = 1

                elif modality_type == "video":
                    t = video_frame_num
                    h = video_grid_thw[video_index][1]
                    w = video_grid_thw[video_index][2]
                    h_int, w_int = int(h), int(w)
                    llm_grid_h = h_int // spatial_merge_size
                    llm_grid_w = w_int // spatial_merge_size

                    for t_idx in range(t):
                        t_index = (
                            torch.tensor(t_idx, device=position_ids.device)
                            .view(-1, 1)
                            .expand(1, llm_grid_h * llm_grid_w)
                            .reshape(-1)
                        )
                        h_index = (
                            torch.arange(llm_grid_h, device=position_ids.device)
                            .view(1, -1, 1)
                            .expand(1, llm_grid_h, llm_grid_w)
                            .reshape(-1)
                        )
                        w_index = (
                            torch.arange(llm_grid_w, device=position_ids.device)
                            .view(1, 1, -1)
                            .expand(1, llm_grid_h, llm_grid_w)
                            .reshape(-1)
                        )
                        llm_pos_ids_list.append(
                            torch.stack([t_index, h_index, w_index]) + st_idx
                        )

                    video_group_index += 1
                    if video_group_index >= video_grid_thw[video_index][0]:
                        video_index += 1
                        video_group_index = 0
                    video_frame_num += 1

                else:  # text
                    text_len = end_idx - start_idx
                    text_range = torch.arange(text_len, device=position_ids.device)
                    text_pos = text_range.view(1, -1).expand(3, text_len) + st_idx
                    llm_pos_ids_list.append(text_pos)
                    video_frame_num = 1

            llm_positions = torch.cat(llm_pos_ids_list, dim=1).reshape(3, -1)
            idx_mask = curr_mask == 1
            position_ids[..., i, idx_mask] = llm_positions.to(position_ids.device)
            mrope_position_deltas.append(
                llm_positions.max() + 1 - len(total_input_ids[i])
            )
        mrope_position_deltas = torch.tensor(
            mrope_position_deltas, device=input_ids.device
        ).unsqueeze(1)
        return position_ids, mrope_position_deltas
    else:
        if attention_mask is not None:
            position_ids = attention_mask.long().cumsum(-1) - 1
            position_ids.masked_fill_(attention_mask == 0, 1)
            position_ids = (
                position_ids.unsqueeze(0).expand(3, -1, -1).to(attention_mask.device)
            )
            max_position_ids = position_ids.amax(dim=0, keepdim=False)
            mrope_position_deltas = (
                max_position_ids.amax(-1, keepdim=True) + 1 - attention_mask.shape[-1]
            )
        else:
            length = input_ids.shape[1]
            batch_size = input_ids.shape[0]
            arange_ids = torch.arange(length, device=input_ids.device).view(1, 1, -1)
            position_ids = arange_ids.expand(3, batch_size, length)
            mrope_position_deltas = torch.zeros(
                [batch_size, 1],
                device=input_ids.device,
                dtype=input_ids.dtype,
            )
        return position_ids, mrope_position_deltas
```
**EN:** This block defines `get_rope_index_glm4v` and contains the main logic for this step. It mainly invokes `torch.ones`, `attention_mask.to`, `enumerate`, `torch.tensor.unsqueeze`, and `torch.ones_like`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `image_token_id`, `video_start_token_id`, `video_end_token_id`, `spatial_merge_size`, and `mrope_position_deltas` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_rope_index_glm4v`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ones`、`attention_mask.to`、`enumerate`、`torch.tensor.unsqueeze` 以及 `torch.ones_like`，说明该流程会编排底层辅助函数或计算内核。 像 `image_token_id`、`video_start_token_id`、`video_end_token_id`、`spatial_merge_size` 以及 `mrope_position_deltas` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 669-827: `get_rope_index_ernie45` getter for RoPE index ernie 45
```python
def get_rope_index_ernie45(
    input_ids: torch.Tensor,
    hf_config: Any,
    image_grid_thw: Union[List[List[int]], torch.Tensor],
    video_grid_thw: Union[List[List[int]], torch.Tensor],
    **kwargs,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Get mrope input positions and delta value for Ernie VL."""
    image_token_id = hf_config.im_patch_id
    video_start_token_id = hf_config.video_start_token_id
    video_end_token_id = hf_config.video_end_token_id
    spatial_conv_size = hf_config.spatial_conv_size
    temporal_conv_size = hf_config.temporal_conv_size

    mrope_position_deltas = []
    if input_ids is not None and (
        image_grid_thw is not None or video_grid_thw is not None
    ):
        total_input_ids = input_ids
        position_ids = torch.ones(
            3,
            input_ids.shape[0],
            input_ids.shape[1],
            dtype=input_ids.dtype,
            device=input_ids.device,
        )
        image_index, video_index = 0, 0
        for i, input_ids in enumerate(total_input_ids):
            input_tokens = input_ids.tolist()

            input_token_type = []
            video_check_flg = False
            for token in input_tokens:
                if token == video_start_token_id:
                    video_check_flg = True
                elif token == video_end_token_id:
                    video_check_flg = False

                if token == image_token_id and not video_check_flg:
                    input_token_type.append("image")
                elif token == image_token_id and video_check_flg:
                    input_token_type.append("video")
                else:
                    input_token_type.append("text")

            input_type_group = []
            for key, group in itertools.groupby(
                enumerate(input_token_type), lambda x: x[1]
            ):
                group = list(group)
                start_index = group[0][0]
                end_index = group[-1][0] + 1
                input_type_group.append((key, start_index, end_index))

            llm_pos_ids_list = []
            video_frame_num = 1
            for modality_type, start_idx, end_idx in input_type_group:
                st_idx = (
                    llm_pos_ids_list[-1].max() + 1 if len(llm_pos_ids_list) > 0 else 0
                )

                if modality_type == "image":
                    t, h, w = (
                        image_grid_thw[image_index][0],
                        image_grid_thw[image_index][1],
                        image_grid_thw[image_index][2],
                    )
                    llm_grid_t, llm_grid_h, llm_grid_w = (
                        t.item(),
                        h.item() // spatial_conv_size,
                        w.item() // spatial_conv_size,
                    )

                    t_index = (
                        torch.arange(llm_grid_t)
                        .view(-1, 1)
                        .expand(-1, llm_grid_h * llm_grid_w)
                        .flatten()
                    )
                    h_index = (
                        torch.arange(llm_grid_h)
                        .view(1, -1, 1)
                        .expand(llm_grid_t, -1, llm_grid_w)
                        .flatten()
                    )
                    w_index = (
                        torch.arange(llm_grid_w)
                        .view(1, 1, -1)
                        .expand(llm_grid_t, llm_grid_h, -1)
                        .flatten()
                    )
                    llm_pos_ids_list.append(
                        torch.stack([t_index, h_index, w_index]) + st_idx
                    )
                    image_index += 1
                    video_frame_num = 1

                elif modality_type == "video":
                    t, h, w = (
                        video_grid_thw[video_index][0],
                        video_grid_thw[video_index][1],
                        video_grid_thw[video_index][2],
                    )
                    llm_grid_t, llm_grid_h, llm_grid_w = (
                        t.item() // temporal_conv_size,
                        h.item() // spatial_conv_size,
                        w.item() // spatial_conv_size,
                    )

                    for t_idx in range(llm_grid_t):
                        t_index = (
                            torch.tensor(t_idx)
                            .view(-1, 1)
                            .expand(-1, llm_grid_h * llm_grid_w)
                            .flatten()
                        )
                        h_index = (
                            torch.arange(llm_grid_h)
                            .view(1, -1, 1)
                            .expand(1, -1, llm_grid_w)
                            .flatten()
                        )
                        w_index = (
                            torch.arange(llm_grid_w)
                            .view(1, 1, -1)
                            .expand(1, llm_grid_h, -1)
                            .flatten()
                        )
                        llm_pos_ids_list.append(
                            torch.stack([t_index, h_index, w_index]) + st_idx
                        )
                    video_index += 1
                    video_frame_num += 1

                else:
                    text_len = end_idx - start_idx
                    llm_pos_ids_list.append(
                        torch.arange(text_len).view(1, -1).expand(3, -1) + st_idx
                    )
                    video_frame_num = 1

            llm_positions = torch.cat(llm_pos_ids_list, dim=1).reshape(3, -1)
            position_ids[..., i, :] = llm_positions.to(position_ids.device)
            mrope_position_deltas.append(
                llm_positions.max() + 1 - len(total_input_ids[i])
            )
        mrope_position_deltas = torch.tensor(
            mrope_position_deltas, device=input_ids.device
        ).unsqueeze(1)
        return position_ids, mrope_position_deltas
    else:
        s = input_ids.shape[1]
        position_ids = torch.arange(s)
        position_ids = position_ids.unsqueeze(0).expand(3, -1, -1).to(input_ids.device)
        max_position_ids = position_ids.max(0, keepdim=False)[0].max(-1, keepdim=True)[
            0
        ]
        mrope_position_deltas = max_position_ids + 1 - s
        return position_ids, mrope_position_deltas
```
**EN:** This block defines `get_rope_index_ernie45` and contains the main logic for this step. It mainly invokes `torch.ones`, `enumerate`, `torch.tensor.unsqueeze`, `torch.arange`, and `position_ids.unsqueeze.expand.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `image_token_id`, `video_start_token_id`, `video_end_token_id`, `spatial_conv_size`, and `temporal_conv_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_rope_index_ernie45`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ones`、`enumerate`、`torch.tensor.unsqueeze`、`torch.arange` 以及 `position_ids.unsqueeze.expand.to`，说明该流程会编排底层辅助函数或计算内核。 像 `image_token_id`、`video_start_token_id`、`video_end_token_id`、`spatial_conv_size` 以及 `temporal_conv_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_get_feat_extract_output_lengths`, `_get_llm_pos_ids_for_vision`, `get_rope_index`, `get_rope_index_qwen3_omni`, and `get_rope_index_glm4v`. / **主要符号**：核心入口包括 `_get_feat_extract_output_lengths`、`_get_llm_pos_ids_for_vision`、`get_rope_index`、`get_rope_index_qwen3_omni` 以及 `get_rope_index_glm4v`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `itertools`, `typing.Any`, `typing.List`, `typing.Optional`, `typing.Tuple`, and `typing.Union` / **标准库**：`__future__.annotations`、`itertools`、`typing.Any`、`typing.List`、`typing.Optional`、`typing.Tuple` 以及 `typing.Union`
- **Third-party**: `torch` / **第三方依赖**：`torch`
