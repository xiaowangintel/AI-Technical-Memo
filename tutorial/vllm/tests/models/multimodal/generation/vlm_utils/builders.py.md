# builders.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/vlm_utils/builders.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for multimodal processing and model-facing behavior. The file exposes 11 helper/class block(s) used by nearby tests. / [CN] 为多模态处理与面向模型的行为提供共享测试工具。该文件暴露了 11 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L32)
```python
from collections.abc import Callable, Iterable
from pathlib import PosixPath
from typing import Any

import numpy.typing as npt
import torch

from vllm.multimodal.audio import AudioResampler
from vllm.multimodal.image import rescale_image_size
from vllm.multimodal.video import (
    rescale_video_size,
    resize_video,
    sample_frames_from_video,
)

from .....conftest import AudioTestAssets, ImageTestAssets, VideoTestAssets
from .types import (
    SINGLE_AUDIO_BASE_PROMPT,
# ... 6 lines omitted for brevity ...
    PromptWithMultiModalInput,
    SizeType,
    VLMTestInfo,
)
```
**EN:** Imports standard-library modules such as `collections.abc.Callable`, `collections.abc.Iterable`, `pathlib.PosixPath`, third-party packages like `numpy.typing`, `torch`, project helpers such as `vllm.multimodal.audio.AudioResampler`, `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.video.rescale_video_size`.
**CN:** 导入标准库模块（如 `collections.abc.Callable`、`collections.abc.Iterable`、`pathlib.PosixPath`）、第三方包（如 `numpy.typing`、`torch`）、项目内辅助模块（如 `vllm.multimodal.audio.AudioResampler`、`vllm.multimodal.image.rescale_image_size`、`vllm.multimodal.video.rescale_video_size`）。

### Helper / 辅助函数: replace_test_placeholder (L35-L46)
```python
def replace_test_placeholder(
    prompt: str, mm_idx_to_prompt: Callable[[int], str], test_placeholder: str
) -> str:
    """Given a prompt, replaces each test placeholder with the
    model-specific tag.
    """
    prompt_segments = prompt.split(test_placeholder)
    img_prompt = prompt_segments[0]
    for placeholder_idx, next_seg in enumerate(prompt_segments[1:], start=1):
        img_prompt += mm_idx_to_prompt(placeholder_idx)
        img_prompt += next_seg
    return img_prompt
```
**EN:** This helper encapsulates reusable logic in `replace_test_placeholder`. Key inputs are `prompt`, `mm_idx_to_prompt`, `test_placeholder`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `replace_test_placeholder` 中。 关键输入包括 `prompt`、`mm_idx_to_prompt`、`test_placeholder`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_model_prompts (L49-L90)
```python
def get_model_prompts(
    base_prompts: Iterable[str],
    img_idx_to_prompt: Callable[[int], str] | None,
    video_idx_to_prompt: Callable[[int], str] | None,
    audio_idx_to_prompt: Callable[[int], str] | None,
    prompt_formatter: Callable[[str], str],
) -> list[str]:
    """Given a model-agnostic base prompt and test configuration for a model(s)
    to be tested, update the media placeholders and apply the prompt formatting
    to get the test prompt string for this model.

    Example for phi3v, given the base_prompt: "<image>What is the season?"
        1. Replace img placeholder(s)
          -> "<|image_1|>\nWhat is the season?"
        2. Apply prompt formatter:
          -> <|user|>\n<|image_1|>\nWhat is the season?<|end|>\n<|assistant|>\n
    """
    assert isinstance(base_prompts, (list, tuple))
# ... 16 lines omitted for brevity ...
                base_prompt, audio_idx_to_prompt, TEST_AUDIO_PLACEHOLDER
            )

        # Apply the prompt formatter to wrap the base prompt with
        # the correct media placeholders to get the model test prompt
        model_prompt = prompt_formatter(base_prompt)
        model_prompts.append(model_prompt)
    return model_prompts
```
**EN:** This helper encapsulates reusable logic in `get_model_prompts`. Key inputs are `base_prompts`, `img_idx_to_prompt`, `video_idx_to_prompt`, `audio_idx_to_prompt`, `prompt_formatter`. It returns computed state or helper objects back to the caller. The main assertion is `isinstance(base_prompts, (list, tuple))`.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_model_prompts` 中。 关键输入包括 `base_prompts`、`img_idx_to_prompt`、`video_idx_to_prompt`、`audio_idx_to_prompt`、`prompt_formatter`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `isinstance(base_prompts, (list, tuple))`。

### Helper / 辅助函数: build_single_image_inputs_from_test_info (L93-L123)
```python
def build_single_image_inputs_from_test_info(
    test_info: VLMTestInfo,
    image_assets: ImageTestAssets,
    size_wrapper: ImageSizeWrapper,
    tmp_path: PosixPath | None = None,
) -> list[PromptWithMultiModalInput]:
    if test_info.prompt_formatter is None:
        raise ValueError("Prompt formatter must be set to build single image inputs")

    model_prompts = get_model_prompts(
        test_info.single_image_prompts,
        test_info.img_idx_to_prompt,
        test_info.video_idx_to_prompt,
        test_info.audio_idx_to_prompt,
        test_info.prompt_formatter,
    )

    # For models that require a local path / URL encoded in the image; export
# ... 5 lines omitted for brevity ...
        model_prompts = [
            test_info.prompt_path_encoder(tmp_path, prompt, [asset])
            for prompt, asset in zip(model_prompts, image_assets)
        ]

    images = [asset.pil_image for asset in image_assets]
    assert len(images) == len(model_prompts)
    return build_single_image_inputs(images, model_prompts, size_wrapper)
```
**EN:** This helper encapsulates reusable logic in `build_single_image_inputs_from_test_info`. Key inputs are `test_info`, `image_assets`, `size_wrapper`, `tmp_path`. It returns computed state or helper objects back to the caller. The main assertion is `len(images) == len(model_prompts)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_single_image_inputs_from_test_info` 中。 关键输入包括 `test_info`、`image_assets`、`size_wrapper`、`tmp_path`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `len(images) == len(model_prompts)`。

### Helper / 辅助函数: build_single_image_inputs (L126-L144)
```python
def build_single_image_inputs(
    images, model_prompts, size_wrapper: ImageSizeWrapper
) -> list[PromptWithMultiModalInput]:
    # For every image / prompt pair, get a pair containing two lists of
    # length size_factors, where the first contains duplicates of the model
    # prompt [str], and the second contains copies of the image after being
    # scaled by one of the size factors.
    #
    # NOTE: rescaling preserves the image aspect ratio.
    return [
        PromptWithMultiModalInput(
            prompts=[prompt for _ in size_wrapper.data],
            image_data=[
                apply_image_size_scaling(image, size, size_wrapper.type)
                for size in size_wrapper.data
            ],
        )
        for image, prompt in zip(images, model_prompts)
    ]
```
**EN:** This helper encapsulates reusable logic in `build_single_image_inputs`. Key inputs are `images`, `model_prompts`, `size_wrapper`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_single_image_inputs` 中。 关键输入包括 `images`、`model_prompts`、`size_wrapper`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: build_multi_image_inputs_from_test_info (L147-L179)
```python
def build_multi_image_inputs_from_test_info(
    test_info: VLMTestInfo,
    image_assets: ImageTestAssets,
    size_wrapper: ImageSizeWrapper,
    tmp_path: PosixPath | None = None,
) -> list[PromptWithMultiModalInput]:
    if test_info.prompt_formatter is None:
        raise ValueError("Prompt formatter must be set to build multi image inputs")

    model_prompts = get_model_prompts(
        [test_info.multi_image_prompt],
        test_info.img_idx_to_prompt,
        test_info.video_idx_to_prompt,
        test_info.audio_idx_to_prompt,
        test_info.prompt_formatter,
    )

    if test_info.prompt_path_encoder is not None:
# ... 7 lines omitted for brevity ...
    images = [asset.pil_image for asset in image_assets]

    # Currently, we only have one multi-image list & one multi-image prompt
    return build_multi_image_inputs(
        image_lists=[images],
        model_prompts=model_prompts,
        size_wrapper=size_wrapper,
    )
```
**EN:** This helper encapsulates reusable logic in `build_multi_image_inputs_from_test_info`. Key inputs are `test_info`, `image_assets`, `size_wrapper`, `tmp_path`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_multi_image_inputs_from_test_info` 中。 关键输入包括 `test_info`、`image_assets`、`size_wrapper`、`tmp_path`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: build_multi_image_inputs (L182-L197)
```python
def build_multi_image_inputs(
    image_lists, model_prompts, size_wrapper: ImageSizeWrapper
) -> list[PromptWithMultiModalInput]:
    return [
        PromptWithMultiModalInput(
            prompts=[prompt for _ in size_wrapper.data],
            image_data=[
                [
                    apply_image_size_scaling(image, size, size_wrapper.type)
                    for image in images
                ]
                for size in size_wrapper.data
            ],
        )
        for images, prompt in zip(image_lists, model_prompts)
    ]
```
**EN:** This helper encapsulates reusable logic in `build_multi_image_inputs`. Key inputs are `image_lists`, `model_prompts`, `size_wrapper`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_multi_image_inputs` 中。 关键输入包括 `image_lists`、`model_prompts`、`size_wrapper`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: build_embedding_inputs_from_test_info (L200-L233)
```python
def build_embedding_inputs_from_test_info(
    test_info: VLMTestInfo,
    image_assets: ImageTestAssets,
    size_wrapper: ImageSizeWrapper,
):
    # These conditions will always be true if invoked through filtering,
    # but we still check them in case this is ever called directly
    if test_info.prompt_formatter is None:
        raise ValueError("Prompt formatter must be set to build image embedding inputs")
    if size_wrapper.type != SizeType.SIZE_FACTOR or not all(
        factor == 1.0 for factor in size_wrapper.data
    ):
        raise ValueError("Embedding tests require constant (1.0) size factors")
    if test_info.convert_assets_to_embeddings is None:
        raise ValueError("No conversion func for getting embeddings found")

    model_prompts = get_model_prompts(
        SINGLE_IMAGE_BASE_PROMPTS,
# ... 8 lines omitted for brevity ...
    if test_info.dtype != "auto":
        dtype = getattr(torch, test_info.dtype)  # type: ignore
        embeds = [e.to(dtype=dtype) for e in embeds]
    assert len(images) == len(model_prompts)

    inputs = build_single_image_inputs(images, model_prompts, size_wrapper)
    vllm_embeddings = build_single_image_inputs(embeds, model_prompts, size_wrapper)
    return inputs, vllm_embeddings
```
**EN:** This helper encapsulates reusable logic in `build_embedding_inputs_from_test_info`. Key inputs are `test_info`, `image_assets`, `size_wrapper`. It returns computed state or helper objects back to the caller. The main assertion is `len(images) == len(model_prompts)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_embedding_inputs_from_test_info` 中。 关键输入包括 `test_info`、`image_assets`、`size_wrapper`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `len(images) == len(model_prompts)`。

### Helper / 辅助函数: build_video_inputs_from_test_info (L236-L278)
```python
def build_video_inputs_from_test_info(
    test_info: VLMTestInfo,
    video_assets: VideoTestAssets,
    size_wrapper: ImageSizeWrapper,
    num_frames: int,
    needs_video_metadata: bool,
) -> list[PromptWithMultiModalInput]:
    if test_info.prompt_formatter is None:
        raise ValueError("Prompt formatter must be set to build video inputs")
    model_prompts = get_model_prompts(
        [VIDEO_BASE_PROMPT],
        test_info.img_idx_to_prompt,
        test_info.video_idx_to_prompt,
        test_info.audio_idx_to_prompt,
        test_info.prompt_formatter,
    )

    sampled_vids = [
# ... 17 lines omitted for brevity ...
                    if not needs_video_metadata
                    else (video_scaler(video, size), meta)
                )
                for size in size_wrapper.data
            ],
        )
        for (video, meta), prompt in zip(sampled_vids, model_prompts)
    ]
```
**EN:** This helper encapsulates reusable logic in `build_video_inputs_from_test_info`. Key inputs are `test_info`, `video_assets`, `size_wrapper`, `num_frames`, `needs_video_metadata`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_video_inputs_from_test_info` 中。 关键输入包括 `test_info`、`video_assets`、`size_wrapper`、`num_frames`、`needs_video_metadata`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: sample_frames_with_video_metadata (L281-L292)
```python
def sample_frames_with_video_metadata(
    video_with_meta: tuple[npt.NDArray, dict[str, Any]],
    num_frames: int,
) -> tuple[npt.NDArray, dict[str, Any]]:
    video, meta = video_with_meta
    video = sample_frames_from_video(video, num_frames)

    meta["do_sample_frames"] = meta["total_num_frames"] == num_frames
    meta["total_num_frames"] = num_frames
    meta["fps"] = meta["duration"] / num_frames
    meta["frames_indices"] = list(range(num_frames))
    return video, meta
```
**EN:** This helper encapsulates reusable logic in `sample_frames_with_video_metadata`. Key inputs are `video_with_meta`, `num_frames`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `sample_frames_with_video_metadata` 中。 关键输入包括 `video_with_meta`、`num_frames`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: apply_image_size_scaling (L295-L310)
```python
def apply_image_size_scaling(image, size: float | tuple[int, int], size_type: SizeType):
    """Applies a size scaler to one image; this can be an image size factor,
    which scales the image while maintaining the aspect ratio"""
    # Special case for embeddings; if it's a tensor, it's only valid if we
    # are considering size factors at constant scale, i.e., we just clone
    # the tensor
    if isinstance(image, torch.Tensor):
        assert size_type == SizeType.SIZE_FACTOR and size == 1
        return image
    if size_type == SizeType.SIZE_FACTOR:
        # We have a list of image size factors
        return rescale_image_size(image, size)
    elif size_type == SizeType.FIXED_SIZE:
        # We have a list of fixed sizes
        return image.resize(size)
    raise ValueError("ImageSizeWrapper type must be FIXED_SIZE or SIZE_FACTOR")
```
**EN:** This helper encapsulates reusable logic in `apply_image_size_scaling`. Key inputs are `image`, `size`, `size_type`. It returns computed state or helper objects back to the caller. The main assertion is `size_type == SizeType.SIZE_FACTOR and size == 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `apply_image_size_scaling` 中。 关键输入包括 `image`、`size`、`size_type`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `size_type == SizeType.SIZE_FACTOR and size == 1`。

### Helper / 辅助函数: build_audio_inputs_from_test_info (L313-L344)
```python
def build_audio_inputs_from_test_info(
    test_info: VLMTestInfo,
    audio_assets: AudioTestAssets,
) -> list[PromptWithMultiModalInput]:
    if test_info.prompt_formatter is None:
        raise ValueError("Prompt formatter must be set to build audio inputs")
    model_prompts = get_model_prompts(
        SINGLE_AUDIO_BASE_PROMPT,
        test_info.img_idx_to_prompt,
        test_info.video_idx_to_prompt,
        test_info.audio_idx_to_prompt,
        test_info.prompt_formatter,
    )
    resampler = AudioResampler(target_sr=16000)
    audios = [asset.audio_and_sample_rate for asset in audio_assets]
    resampled_audios = [
        (
            resampler.resample(
# ... 6 lines omitted for brevity ...
    ]

    return [
        PromptWithMultiModalInput(
            prompts=model_prompts,
            audio_data=resampled_audios,
        )
    ]
```
**EN:** This helper encapsulates reusable logic in `build_audio_inputs_from_test_info`. Key inputs are `test_info`, `audio_assets`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_audio_inputs_from_test_info` 中。 关键输入包括 `test_info`、`audio_assets`。 它把计算得到的状态或辅助对象返回给调用方。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Callable`, `collections.abc.Iterable`, `pathlib.PosixPath`, `typing.Any`
- **Third-party / 第三方**: `numpy.typing`, `torch`
- **Project / 项目内**: `vllm.multimodal.audio.AudioResampler`, `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.video.rescale_video_size`, `vllm.multimodal.video.resize_video`, `vllm.multimodal.video.sample_frames_from_video`
- **Local relative imports / 本地相对导入**: `.....conftest.AudioTestAssets`, `.....conftest.ImageTestAssets`, `.....conftest.VideoTestAssets`, `.types.ImageSizeWrapper`, `.types.PromptWithMultiModalInput`, `.types.SINGLE_AUDIO_BASE_PROMPT`, `.types.SINGLE_IMAGE_BASE_PROMPTS`, `.types.SizeType`, `.types.TEST_AUDIO_PLACEHOLDER`, `.types.TEST_IMG_PLACEHOLDER`, `.types.TEST_VIDEO_PLACEHOLDER`, `.types.VIDEO_BASE_PROMPT`, `.types.VLMTestInfo`
