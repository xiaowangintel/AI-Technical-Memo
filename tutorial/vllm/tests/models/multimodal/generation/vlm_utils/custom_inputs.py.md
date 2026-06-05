# custom_inputs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/vlm_utils/custom_inputs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for multimodal processing and model-facing behavior. The file exposes 5 helper/class block(s) used by nearby tests. / [CN] 为多模态处理与面向模型的行为提供共享测试工具。该文件暴露了 5 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L17)
```python
from collections.abc import Callable

from vllm.assets.image import ImageAsset
from vllm.multimodal.image import rescale_image_size
from vllm.multimodal.video import (
    rescale_video_size,
    resize_video,
    sample_frames_from_video,
)

from .....conftest import IMAGE_ASSETS, VIDEO_ASSETS
from .builders import build_multi_image_inputs, build_single_image_inputs
from .types import ImageSizeWrapper, PromptWithMultiModalInput, SizeType
```
**EN:** Imports standard-library modules such as `collections.abc.Callable`, project helpers such as `vllm.assets.image.ImageAsset`, `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.video.rescale_video_size`.
**CN:** 导入标准库模块（如 `collections.abc.Callable`）、项目内辅助模块（如 `vllm.assets.image.ImageAsset`、`vllm.multimodal.image.rescale_image_size`、`vllm.multimodal.video.rescale_video_size`）。

### Helper / 辅助函数: multi_image_multi_aspect_ratio_inputs (L20-L58)
```python
def multi_image_multi_aspect_ratio_inputs(formatter: Callable[[str], str]):
    """Builds inputs for multi-image (varied sizes/aspect ratio) testing.

    Args:
        formatter: model-specific prompt formatter.
    """
    stop_sign = IMAGE_ASSETS[0].pil_image
    cherry_blossom = IMAGE_ASSETS[1].pil_image

    # Apply the selected formatter to the base prompts
    img_prompts = [
        "<image><image>\nDescribe 2 images.",
        "<image><image>\nDescribe 2 images.",
        "<image><image><image><image>\nDescribe 4 images.",
        "<image>\nWhat is the season?",
    ]
    formatted_prompts = [formatter(prompt) for prompt in img_prompts]
    aspect_ratio_images = [
# ... 13 lines omitted for brevity ...
    ]

    return [
        PromptWithMultiModalInput(
            prompts=formatted_prompts,
            image_data=aspect_ratio_images,
        )
    ]
```
**EN:** This helper encapsulates reusable logic in `multi_image_multi_aspect_ratio_inputs`. Key inputs are `formatter`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `multi_image_multi_aspect_ratio_inputs` 中。 关键输入包括 `formatter`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: multi_video_multi_aspect_ratio_inputs (L61-L99)
```python
def multi_video_multi_aspect_ratio_inputs(
    formatter: Callable[[str], str], num_frames: int = 16
):
    """Builds inputs for multi-video (varied sizes/aspect ratio) testing.

    Args:
        formatter: model-specific prompt formatter.
    """
    video = sample_frames_from_video(VIDEO_ASSETS[0].np_ndarrays, num_frames)
    # Apply the selected formatter to the base prompts
    video_prompts = [
        "<video><video>\nDescribe 2 videos.",
        "<video><video>\nDescribe 2 videos.",
        "<video><video><video><video>\nDescribe 4 videos.",
        "<video>\nWhy is this video funny?",
    ]
    formatted_prompts = [formatter(prompt) for prompt in video_prompts]
    aspect_ratio_videos = [
# ... 13 lines omitted for brevity ...
    ]

    return [
        PromptWithMultiModalInput(
            prompts=formatted_prompts,
            video_data=aspect_ratio_videos,
        )
    ]
```
**EN:** This helper encapsulates reusable logic in `multi_video_multi_aspect_ratio_inputs`. Key inputs are `formatter`, `num_frames`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `multi_video_multi_aspect_ratio_inputs` 中。 关键输入包括 `formatter`、`num_frames`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: different_patch_input_cases_internvl (L102-L121)
```python
def different_patch_input_cases_internvl():
    images = [asset.pil_image.resize((896, 896)) for asset in IMAGE_ASSETS]
    formatter = (
        lambda img_prompt: f"<|im_start|>User\n{img_prompt}<|im_end|>\n<|im_start|>Assistant\n"  # noqa: E501
    )
    single_img_prompts = [
        "<image>\nWhat's the content in the center of the image?",
        "<image>\nWhat is the season?",
    ]
    multi_img_prompts = [
        "Image-1: <image>\nImage-2: <image>\nDescribe the two images in detail.\n",  # noqa: E501
    ]
    formatted_sprompts = [formatter(prompt) for prompt in single_img_prompts]
    formatted_mprompts = [formatter(prompt) for prompt in multi_img_prompts]

    wrapped_sf = ImageSizeWrapper(type=SizeType.SIZE_FACTOR, data=[0.5, 1.0])
    return [
        build_single_image_inputs(images, formatted_sprompts, wrapped_sf),
        build_multi_image_inputs([images], formatted_mprompts, wrapped_sf),
    ]
```
**EN:** This helper encapsulates reusable logic in `different_patch_input_cases_internvl`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `different_patch_input_cases_internvl` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: windows_attention_image_qwen2_5_vl (L124-L135)
```python
def windows_attention_image_qwen2_5_vl():
    # image from regression issue: https://github.com/vllm-project/vllm/issues/15122 # noqa: E501
    image = ImageAsset("hato").pil_image

    question = "Describe the image."
    img_prompt = "<|vision_start|><|image_pad|><|vision_end|>"
    prompt = (
        f"<|im_start|>User\n{img_prompt}{question}<|im_end|>\n<|im_start|>assistant\n"
    )

    wrapped_sf = ImageSizeWrapper(type=SizeType.SIZE_FACTOR, data=[0.5])
    return build_single_image_inputs([image], [prompt], wrapped_sf)
```
**EN:** This helper encapsulates reusable logic in `windows_attention_image_qwen2_5_vl`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `windows_attention_image_qwen2_5_vl` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: video_with_metadata_glm4_1v (L138-L156)
```python
def video_with_metadata_glm4_1v():
    video_array = VIDEO_ASSETS[0].np_ndarrays
    metadata = VIDEO_ASSETS[0].metadata
    question = "Describe the video."
    video_prompt = "<|begin_of_video|><|video|><|end_of_video|>"
    formatted_prompt = f"[gMASK]<|user|>\n{video_prompt}{question}<|assistant|>\n"

    scales = [0.1, 0.2, 0.25]
    video_input = [
        [(rescale_video_size(video_array, scale), metadata)] for scale in scales
    ]
    prompts = [formatted_prompt] * len(video_input)

    return [
        PromptWithMultiModalInput(
            prompts=prompts,
            video_data=video_input,
        )
    ]
```
**EN:** This helper encapsulates reusable logic in `video_with_metadata_glm4_1v`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `video_with_metadata_glm4_1v` 中。 它把计算得到的状态或辅助对象返回给调用方。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Callable`
- **Project / 项目内**: `vllm.assets.image.ImageAsset`, `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.video.rescale_video_size`, `vllm.multimodal.video.resize_video`, `vllm.multimodal.video.sample_frames_from_video`
- **Local relative imports / 本地相对导入**: `.....conftest.IMAGE_ASSETS`, `.....conftest.VIDEO_ASSETS`, `.builders.build_multi_image_inputs`, `.builders.build_single_image_inputs`, `.types.ImageSizeWrapper`, `.types.PromptWithMultiModalInput`, `.types.SizeType`
