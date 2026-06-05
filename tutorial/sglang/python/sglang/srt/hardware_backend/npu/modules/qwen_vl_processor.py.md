# qwen_vl_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/modules/qwen_vl_processor.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for backend-specific runtime support inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的后端运行时支持支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Module setup and shared state / 模块设置与共享状态
```python
import torch
import torchvision.transforms.v2.functional as tvF
from transformers.image_processing_utils import BatchFeature
from transformers.image_transforms import group_images_by_shape, reorder_images
from transformers.image_utils import (
    ChannelDimension,
    PILImageResampling,
    SizeDict,
    get_image_size,
)
from transformers.models.qwen2_vl.image_processing_qwen2_vl import smart_resize
from transformers.models.qwen3_vl.video_processing_qwen3_vl import (
    smart_resize as smart_resize_video,
)
from transformers.utils import TensorType
from transformers.video_utils import group_videos_by_shape, reorder_videos

from sglang.srt.utils import apply_module_patch
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `torch`, `torchvision.transforms.v2.functional`, `transformers.image_processing_utils`, `transformers.image_transforms`, `transformers.image_utils`, `transformers.models.qwen2_vl.image_processing_qwen2_vl`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `torch`, `torchvision.transforms.v2.functional`, `transformers.image_processing_utils`, `transformers.image_transforms`, `transformers.image_utils`, `transformers.models.qwen2_vl.image_processing_qwen2_vl`。

### Lines 21-58: Function `transform_patches_to_flatten` / 函数 `transform_patches_to_flatten`
```python
def transform_patches_to_flatten(
    patches: torch.Tensor,
    batch_size: int,
    grid_t: int,
    temporal_patch_size: int,
    channel: int,
    grid_h: int,
    grid_w: int,
    patch_size: int,
    merge_size: int,
) -> torch.Tensor:
    patches = patches.view(
        batch_size * grid_t,
        temporal_patch_size * channel,
        grid_h // merge_size,
        merge_size,
        patch_size,
        grid_w // merge_size,
        merge_size,
        patch_size,
    )
    patches = patches.permute(0, 1, 2, 5, 3, 6, 4, 7)
    patches = patches.reshape(
        batch_size,
        grid_t,
        temporal_patch_size,
        channel,
        grid_h * grid_w,
# ... omitted for brevity ...
    flatten_patches = patches.reshape(
        batch_size,
        grid_t * grid_h * grid_w,
        -1,
    )
    return flatten_patches
```
**EN:** This function implements `transform_patches_to_flatten`. It primarily calls `patches.view`, `patches.permute`, `patches.reshape` to complete its work. State updates are written into `patches`, `flatten_patches`.
**CN:** 该函数实现了 `transform_patches_to_flatten`。 它主要通过调用 `patches.view`, `patches.permute`, `patches.reshape` 来完成任务。 状态更新主要写入 `patches`, `flatten_patches`。

### Lines 63-167: Function `npu_wrapper_preprocess` / 函数 `npu_wrapper_preprocess`
```python
def npu_wrapper_preprocess(func):

    def _preprocess(
        self,
        images: list["torch.Tensor"],
        do_resize: bool,
        size: SizeDict,
        resample: "PILImageResampling | tvF.InterpolationMode | int | None",
        do_rescale: bool,
        rescale_factor: float,
        do_normalize: bool,
        image_mean: float | list[float] | None,
        image_std: float | list[float] | None,
        patch_size: int,
        temporal_patch_size: int,
        merge_size: int,
        disable_grouping: bool | None,
        return_tensors: str | TensorType | None,
        **kwargs,
    ):
        # Group images by size for batched resizing
        grouped_images, grouped_images_index = group_images_by_shape(
            images, disable_grouping=disable_grouping
        )
        resized_images_grouped = {}
        for shape, stacked_images in grouped_images.items():
            height, width = stacked_images.shape[-2:]
            if do_resize:
# ... omitted for brevity ...
        return BatchFeature(
            data={"pixel_values": pixel_values, "image_grid_thw": image_grid_thw},
            tensor_type=return_tensors,
        )

    return _preprocess
```
**EN:** This function implements `npu_wrapper_preprocess`. It primarily calls `group_images_by_shape`, `grouped_images.items`, `reorder_images`, `torch.cat`, `torch.tensor`, `BatchFeature` to complete its work. State updates are written into `resized_images_grouped`, `resized_images`, `processed_images_grouped`, `processed_grids`, `processed_images`, `pixel_values`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `npu_wrapper_preprocess`。 它主要通过调用 `group_images_by_shape`, `grouped_images.items`, `reorder_images`, `torch.cat`, `torch.tensor`, `BatchFeature` 来完成任务。 状态更新主要写入 `resized_images_grouped`, `resized_images`, `processed_images_grouped`, `processed_grids`, `processed_images`, `pixel_values`。 实现中使用了条件分支、迭代逻辑。

### Lines 172-284: Function `npu_wrapper_video_preprocess` / 函数 `npu_wrapper_video_preprocess`
```python
def npu_wrapper_video_preprocess(func):

    def _preprocess(
        self,
        videos: list[torch.Tensor],
        do_convert_rgb: bool = True,
        do_resize: bool = True,
        size: SizeDict | None = None,
        resample: "PILImageResampling | tvF.InterpolationMode | int | None" = PILImageResampling.BICUBIC,
        do_rescale: bool = True,
        rescale_factor: float = 1 / 255.0,
        do_normalize: bool = True,
        image_mean: float | list[float] | None = None,
        image_std: float | list[float] | None = None,
        patch_size: int | None = None,
        temporal_patch_size: int | None = None,
        merge_size: int | None = None,
        return_tensors: str | TensorType | None = None,
        **kwargs,
    ):
        grouped_videos, grouped_videos_index = group_videos_by_shape(videos)
        resized_videos_grouped = {}

        for shape, stacked_videos in grouped_videos.items():
            B, T, C, H, W = stacked_videos.shape
            num_frames, height, width = T, H, W
            if do_resize:
                resized_height, resized_width = smart_resize_video(
# ... omitted for brevity ...
            "video_grid_thw": video_grid_thw,
        }

        return BatchFeature(data=data, tensor_type=return_tensors)

    return _preprocess
```
**EN:** This function implements `npu_wrapper_video_preprocess`. It primarily calls `group_videos_by_shape`, `grouped_videos.items`, `reorder_videos`, `torch.cat`, `torch.tensor`, `BatchFeature` to complete its work. State updates are written into `resized_videos_grouped`, `resized_videos`, `processed_videos_grouped`, `processed_grids`, `processed_videos`, `pixel_values_videos`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `npu_wrapper_video_preprocess`。 它主要通过调用 `group_videos_by_shape`, `grouped_videos.items`, `reorder_videos`, `torch.cat`, `torch.tensor`, `BatchFeature` 来完成任务。 状态更新主要写入 `resized_videos_grouped`, `resized_videos`, `processed_videos_grouped`, `processed_grids`, `processed_videos`, `pixel_values_videos`。 实现中使用了条件分支、迭代逻辑。

### Lines 287-287: Constants and shared state / 常量与共享状态
```python
_npu_preprocess_patched = False
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_npu_preprocess_patched`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_npu_preprocess_patched`。

### Lines 290-304: Function `npu_apply_qwen_image_preprocess_patch` / 函数 `npu_apply_qwen_image_preprocess_patch`
```python
def npu_apply_qwen_image_preprocess_patch():
    global _npu_preprocess_patched
    if _npu_preprocess_patched:
        return
    apply_module_patch(
        "transformers.models.qwen2_vl.image_processing_qwen2_vl.Qwen2VLImageProcessor",
        "_preprocess",
        [npu_wrapper_preprocess],
    )
    apply_module_patch(
        "transformers.models.qwen3_vl.video_processing_qwen3_vl.Qwen3VLVideoProcessor",
        "_preprocess",
        [npu_wrapper_video_preprocess],
    )
    _npu_preprocess_patched = True
```
**EN:** This function implements `npu_apply_qwen_image_preprocess_patch`. It primarily calls `apply_module_patch` to complete its work. State updates are written into `_npu_preprocess_patched`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `npu_apply_qwen_image_preprocess_patch`。 它主要通过调用 `apply_module_patch` 来完成任务。 状态更新主要写入 `_npu_preprocess_patched`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `transform_patches_to_flatten`, `npu_wrapper_preprocess`, `npu_wrapper_video_preprocess`, `npu_apply_qwen_image_preprocess_patch`, `_preprocess`, `_preprocess`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.utils`
- **External / 外部依赖**: `torch`, `torchvision.transforms.v2.functional`, `transformers.image_processing_utils`, `transformers.image_transforms`, `transformers.image_utils`, `transformers.models.qwen2_vl.image_processing_qwen2_vl`, `transformers.models.qwen3_vl.video_processing_qwen3_vl`, `transformers.utils`, `transformers.video_utils`
- **Standard library / 标准库**: None / 无
